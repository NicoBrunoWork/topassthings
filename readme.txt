SELECT 
notif.NOT_NOTI_DS_DT as notification_date,
notif.CUS_CUST_ID as cus_cust_id,
case when paym.cus_cust_id_buy is not null then 1 else 0 end as convirtio
FROM
        `meli-bi-data.WHOWNER.BT_NOTIFICATION_CAMPAIGN` notif
        LEFT JOIN `meli-bi-data.WHOWNER.BT_MP_PAY_PAYMENTS` paym
        ON notif.CUS_CUST_ID = paym.cus_cust_id_buy AND notif.SIT_SITE_ID = paym.sit_site_id AND 
        paym.pay_move_date BETWEEN notif.NOT_NOTI_DS_DT and notif.NOT_NOTI_DS_DT + 2
        and paym.tpv_segment = 'Wallet'
      WHERE
        notif.SIT_SITE_ID = 'MLB'
        AND notif.NOT_NOTI_BUSINESS = 'mercadolibre'
        AND notif.NOT_NOTI_DS_DT BETWEEN '2021-01-01'
        AND '2021-01-02'
        AND notif.NOT_NOTI_PATH = 'NOTIFICATION_CAMPAIGN'
        AND notif.NOT_NOTI_EVENT_TYPE_ID = 'shown'
        AND notif.NOT_NOTI_CAMPAIGN_ID IN ('7P-CELL-RECHARGE-TRIGGER-X-VISITS', '7P-FUND-TRIGGER-X-VISITS', 
        '7P-P2P-TRIGGER-X-VISITS', '7P-QR-TRIGGER-X-VISITS', '7P-SERVICES-TRIGGER-X-VISITS', '7P-SVS-TRIGGER-X-VISITS-VIDEO-CONTENT',
        '7P-WALLET-GENERIC-TRIGGER-X-VISITS')
        AND notif.CUS_CUST_ID IS NOT NULL 
        limit 100


---- Todas las push por usuario desde una fecha dada (sería la fecha del deploy)
DROP TABLE IF EXISTS `meli-marketing.MODELLING.7P_PUSHES_SHOWNS_USER`;
CREATE TABLE `meli-marketing.MODELLING.7P_PUSHES_SHOWNS_USER`
OPTIONS(
expiration_timestamp=TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 2 DAY)
) AS
SELECT 
notif.CUS_CUST_ID as cus_cust_id,
notif.NOT_NOTI_DS_DT as notification_date,
CASE
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%DG%' THEN 'DG'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%REC%' or NOT_NOTI_CAMPAIGN_ID LIKE '%CELL-RECHARGE%' THEN 'REC'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%SVS%'  or NOT_NOTI_CAMPAIGN_ID LIKE '%SERVICES%' THEN 'SVS'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%P2P%' THEN 'P2P'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%QR%' THEN 'QR'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%TRA%' THEN 'TRA'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%ANTENA%' THEN 'ANTENA_RECHARGE'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%ALL%' or NOT_NOTI_CAMPAIGN_ID LIKE '%WALLET-GENERIC%' THEN 'ALL'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%FUND%' THEN 'FUND'
    WHEN NOT_NOTI_CAMPAIGN_ID LIKE '%ENC%' THEN 'SURVEY'
    ELSE 'OTHER'
END AS flow,
case when NOT_NOTI_BATCH_ID like '%DESCUENTO' then 1 else 0 end as descuento,
NOT_NOTI_CAMPAIGN_ID,
rank() over (partition by cus_cust_id order by NOT_NOTI_DS_DT asc) as orden_push
FROM
        `meli-bi-data.WHOWNER.BT_NOTIFICATION_CAMPAIGN` notif
      WHERE
        notif.SIT_SITE_ID = 'MLB'
        AND notif.NOT_NOTI_BUSINESS = 'mercadolibre'
        AND notif.NOT_NOTI_DS_DT BETWEEN '2021-01-01' AND '2021-01-01' + 10
        AND notif.NOT_NOTI_PATH = 'NOTIFICATION_CAMPAIGN'
        AND notif.NOT_NOTI_EVENT_TYPE_ID = 'shown'
        AND notif.NOT_NOTI_CAMPAIGN_ID IN ('7P-CELL-RECHARGE-TRIGGER-X-VISITS', '7P-FUND-TRIGGER-X-VISITS', 
        '7P-P2P-TRIGGER-X-VISITS', '7P-QR-TRIGGER-X-VISITS', '7P-SERVICES-TRIGGER-X-VISITS', '7P-SVS-TRIGGER-X-VISITS-VIDEO-CONTENT',
        '7P-WALLET-GENERIC-TRIGGER-X-VISITS')
        AND notif.CUS_CUST_ID IS NOT NULL;
--- Ultima push por usuario, con toda la info asociada a esa push:
SELECT AS VALUE ARRAY_AGG(pushes ORDER BY orden_push DESC LIMIT 1)[OFFSET(0)]
FROM `meli-marketing.MODELLING.7P_PUSHES_SHOWNS_USER` pushes 
GROUP BY cus_cust_id;
