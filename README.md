# -Bicycle-Manufacturer

[Dataset](https://console.cloud.google.com/bigquery?hl=vi&inv=1&invt=Ab1UAQ&project=my-project-sql-464309&ws=!1m5!1m4!4m3!1sadventureworks2019!2sSales!3sSalesOrderDetail)

| Field Name            | Data Type  | Description                                                                 |
|-----------------------|------------|-----------------------------------------------------------------------------|
| SalesOrderID          | INTEGER    | Unique identifier for each sales order.                                     |
| SalesOrderDetailID    | INTEGER    | Unique identifier for each sales order line item.                           |
| CarrierTrackingNumber | STRING     | Tracking number provided by the shipping carrier.                           |
| OrderQty              | INTEGER    | Number of product units ordered in this line item.                          |
| ProductID             | INTEGER    | Identifier for the product being sold.                                      |
| SpecialOfferID        | INTEGER    | Identifier for the discount or special offer applied.                       |
| UnitPrice             | FLOAT      | Price per unit of the product at the time of the order.                     |
| UnitPriceDiscount     | FLOAT      | Discount applied to the product’s unit price.                               |
| LineTotal             | FLOAT      | Total amount for this line item (calculated as OrderQty × UnitPrice - Discount). |
| rowguid               | STRING     | Globally unique identifier (GUID) for the row.                              |
| ModifiedDate          | TIMESTAMP  | The date and time when the record was last updated.                         |
