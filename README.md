# SQL-PROJECT
SQL PROJECT
SELECT DISTINCT VENDOR_CODE, VENDOR_NAME, UNION_CODE, INVOICE_NUMBER, INVOICE_DATE, SUM(DIS_QTY)DIS_QTY, DISPATCH_DATE, 
DIS_VEHICLE_NUM, SUM(SP_REC_LITRES)SP_REC_LITRES, SUM(SP_REC_DAMAGE_LTRS) SP_REC_DAMAGE_LTRS, SUM(SP_SHORTAGE_LTRS) SP_SHORTAGE_LTRS, SUM(DAMAGE_IN_SP)DAMAGE_IN_SP,
SP_REC_VEHICLE, SUM(AWC_DIS_QTY) AWC_DIS_QTY, SUM(AWC_REC_QTY)AWC_REC_QTY, SUM(AWC_REC_DAMAGE_LTRS)AWC_REC_DAMAGE_LTRS, SUM(nvl(AWC_PENDING_REC_QTY,0))AWC_PENDING_REC_QTY

FROM (SELECT A.VENDOR_CODE, A.VENDOR_NAME, A.UNION_CODE, A.INVOICE_NUMBER, A.INVOICE_DATE, A.BATCH_NO, 
                A.DIS_QTY, A.DISPATCH_DATE,DIS_VEHICLE_NUM,
                NVL(B.SP_REC_LITRES,0)SP_REC_LITRES, 
                NVL(B.SP_REC_DAMAGE_LTRS,0)SP_REC_DAMAGE_LTRS,NVL(B.SP_shortage_ltrs,0)SP_shortage_ltrs,NVL(DAMAGE_IN_SP,0)DAMAGE_IN_SP,
                NVL(SP_REC_VEHICLE,'NOT_RECEIVED_YET')SP_REC_VEHICLE,
                NVL(AWC_DIS_QTY,0)AWC_DIS_QTY,NVL(AWC_REC_QTY,0)AWC_REC_QTY,NVL(AWC_REC_DAMAGE_LTRS,0)AWC_REC_DAMAGE_LTRS,
                (AWC_DIS_QTY-AWC_REC_QTY)AWC_PENDING_REC_QTY
            FROM
                (       (select 1 VENDOR_CODE,'KMF' VENDOR_NAME,UNION_CODE,INVOICE_NUMBER,INVOICE_DATE,batch_no,
                        sum(NET_QTY_LTRS)dis_qty,to_char(INSERTED_DATE,'DD-MM-YYYY') dispatch_date,VEHICLE_NUMBER DIS_VEHICLE_NUM
                        from SAP_KMF_DISPATCH_DETAILS 
                        where month=12 and year=2024 AND to_char(INSERTED_DATE,'DD-MM-YYYY')<>'01-12-2024'
                        group by UNION_CODE,to_char(INSERTED_DATE,'DD-MM-YYYY') ,INVOICE_NUMBER,INVOICE_DATE,batch_no,VEHICLE_NUMBER
                        UNION ALL
                        SELECT 2 VENDOR_CODE,'MEGHANA' VENDOR_NAME,union_code,invoice_number,invoice_date,batch_no,
                        sum(NET_QTY_LTRS)dis_qty,to_char(INSERTED_DATE,'DD-MM-YYYY') dispatch_date,VEHICLE_NUMBER DIS_VEHICLE_NUM
                        FROM UNION_DISPATCH_DETAILS_TBL
                        where month=12 and year=2024 AND to_char(INSERTED_DATE,'DD-MM-YYYY')<>'01-12-2024'
                        group by UNION_CODE,to_char(INSERTED_DATE,'DD-MM-YYYY') ,INVOICE_NUMBER,INVOICE_DATE,batch_no,VEHICLE_NUMBER) A 
                LEFT JOIN 
                        (SELECT  batch_no,SUM(NET_QTY_LTRS)SP_REC_LITRES, SUM(NVL((DAMAGE_LTRS*PACK_QTY_ML)/1000,0))SP_REC_DAMAGE_LTRS,SUM(NVL(shortage_ltrs,0))SP_shortage_ltrs,
                        VEHICLE_NUMBER SP_REC_VEHICLE
                        FROM  SAP_DIST_STOCK_RECEIVE_DETAIL
                        WHERE month=12 AND YEAR=2024
                        GROUP BY batch_no,VEHICLE_NUMBER
                        UNION ALL
                        SELECT  batch_no,SUM(REC_NET_QTY_LTRS)SP_REC_LITRES, SUM(NVL(damage_ltrs,0))SP_REC_DAMAGE_LTRS,SUM(NVL(shortage_ltrs,0))SP_shortage_ltrs,VEHICLE_NUMBER SP_REC_VEHICLE
                        FROM UNION_DISPATCH_DETAILS_TBL
                        WHERE month=12 AND YEAR=2024
                        GROUP BY batch_no,VEHICLE_NUMBER)B
                              ON A.batch_no=B.batch_no 
                LEFT JOIN
                        (SELECT BATCH_NO, SUM(NET_QTY_LTRS)AWC_DIS_QTY FROM SAP_DIST_STOCK_DISPATCH_DETAIL WHERE month=12  AND YEAR=2024 GROUP BY BATCH_NO
                        UNION ALL
                        SELECT BATCH_NO, SUM(NET_QTY_LTRS) AWC_DIS_QTY FROM AWC_STOCK_DIS_TBL  WHERE month=12  AND YEAR=2024 GROUP BY BATCH_NO) C ON b.batch_no=c.batch_no
                        LEFT JOIN
                        (SELECT BATCH_NO,SUM(NET_QTY_LTRS)AWC_REC_QTY,SUM(damaged_ltrs)AWC_REC_DAMAGE_LTRS FROM sap_awc_stock_details WHERE MONTH=12 AND YEAR=2024 AND vendor_code=1 GROUP BY BATCH_NO
                        UNION ALL
                        SELECT BATCH_NO,SUM(NET_QTY_LTRS)AWC_REC_QTY,SUM(damaged_ltrs)AWC_REC_DAMAGE_LTRS FROM sap_awc_stock_details WHERE MONTH=12 AND YEAR=2024 AND vendor_code=3 GROUP BY BATCH_NO)D
                              ON c.batch_no=d.batch_no
                LEFT JOIN 
(SELECT BATCH_NO, SUM(NET_QTY_LTRS)DAMAGE_IN_SP FROM sap_dist_stock_damaged_detail WHERE MONTH=12 AND YEAR=2024 GROUP BY BATCH_NO) E 
                              ON a.batch_no=e.batch_no
))
WHERE DISPATCH_DATE<>'01-12-2024' GROUP BY VENDOR_CODE, VENDOR_NAME, UNION_CODE, INVOICE_NUMBER, INVOICE_DATE,DIS_VEHICLE_NUM,SP_REC_VEHICLE,DISPATCH_DATE;
