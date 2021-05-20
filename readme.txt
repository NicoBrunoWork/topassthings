    762470459 NICO
    680922738
    762467208
    680920948
    
    users_with_3_sends = users_to_send_discounts[users_to_send_discounts['orden_push'] == 3]
    users_with_3_send_descuento = users_with_3_sends[users_with_3_sends['descuento'] == 1] # Con descuento
    users_with_3_send_descuento['prediction_model'] = users_with_3_send_descuento['prediction_model'].apply(change_flow_name)
    users_df = users_df.append(users_with_3_send_descuento)
    users_df = users_df.append(users_with_3_sends[users_with_3_sends['descuento'] == 0]) # Sin descuento
