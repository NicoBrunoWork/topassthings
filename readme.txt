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
