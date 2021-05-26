campaigns_orders = get_model_by_weekday()
print("Orden de la corrida: {}".format(campaigns_orders))
## Aca trae todos los usuarios que matchearon el trigger, que todavía no recibieron el flujo correspondiente y que visitaron el sitio en la última hora
        
users_with_visits = get_non_sent_campaigns_from_trigger(HOUR_VISITS, SIT_SITE_ID, MIN_DECILE, MAX_DECILE, last_digit)
if SIT_SITE_ID != "MLB":
    users_with_visits = users_with_visits[~users_with_visits["prediction_model"].isin(["FUND"])]
    users_with_visits['sent_hour'] = (datetime.utcnow() - timedelta(hours=3)).strftime('%H')
        
    ##Para listar los objetos s3.list_objects_v2(Bucket="fury-data-apps", Prefix="fury_melimarketinguseranalysis/data")['Contents']
        
    thompson = ThompsonSampling(SIT_SITE_ID, ENVIO, MODEL_EQUALS_FLOW)
        
    # Itero sobre los flujos que van rotando
    for campaign in campaigns_orders:
        model_users = users_with_visits[users_with_visits.prediction_model == campaign]
        if model_users.empty:
            slack_error_w_message(MODELO, SITE_ID, f'No hay usuarios con visits para {campaign}')
            continue
        if DISCOUNT_ENABLED:
            print(f'Aplicando descuento para el flujo {campaign}')
            apply_discount(model_users, campaign)
            # Thompson Sampling
            model_users = thompson.assign_wordings_all(model_users)
            normalize_flow(model_users)

        # Primero guardamos los envios y despues los hacemos, por si llegan a demorar mas de 1 hora
        print(f"Actualizando archivo con: {model_users.cust_id.count()} para el flujo {campaign}")
        update_notification_sents_today(model_users, ENVIO, SIT_SITE_ID)

        # Envios
        print(f"Enviando: {model}-{model_users.cust_id.count()}")
        results = send_push_alt(concat_test_users_df(model_users, SIT_SITE_ID, campaign, thompson) , SIT_SITE_ID)
        print(f"Enviados: {len(results)},\nhora: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')},\nflujo: {campaign}")
        print("-"*20)

        # Saco los users que ya le mandé
        users_with_visits = users_with_visits[~users_with_visits.cust_id.isin(model_users.cust_id)]
