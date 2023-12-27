The model pulls the data from the AWS Bulk API into a PowerBI instance running locally on the workstation or in the M365 ecosystem.  It performs some light ETL in PowerQuery/M to derive consistent filterable numeric values for values such as memory, GPU, and memory.  In DAX, helper tables are implemented to facilitate numeric filtering of Network and EBS throughput. Finally, there are simple DAX calculations to derive a three year value for each SKU, which are then rolled up and averaged:

```vb
3yr hourly = IF(
  [Unit]="Hrs",
  [PricePerUnit]*24*365.25*3,
  BLANK())

3yr upfront = IF(
  [Unit]="Quantity",
    (IF(
      [LeaseContractLength]="1yr",
      [PricePerUnit]*3,
      [PricePerUnit])),
  BLANK())

Avg Offer Price (3yr) =
  AVERAGE(ec2_instances[3yr hourly])
  +
  AVERAGE(ec2_instances[3yr upfront])
```
I primarily use this as a backend dataset for Excel queries - this is basically a clean, predictable, relatively fresh set of reference data.  But I've also published this dataset to a [dashboard](https://app.powerbi.com/view?r=eyJrIjoiYzRmOTY1MDYtZmE1ZC00MzA5LWFhMjYtMTIzM2Q0MWMwYjBlIiwidCI6ImZlNGQ5NDA3LWE5NzEtNDhjMy1hOTkzLTRjMmNiOGQ2MjM4NCIsImMiOjF9) where the user can quickly get to a right-fit solution by filtering on resource parameters, such as vCPU, memory, and network bandwith, and consumption model, such as 1 or 3 year commits with variable upfront payments.  I've also added logic that allows the user can filter out solutions lacking guaranteed EBS or Network throughput.  

The table shows a hierarchical view of skus that meet the filter requirements.  The average 3 year price is the 3 year total cost averaged across all the offers associated with the sku.  The numnber of offers corresponds to the number of different consumption models the sku is available under.  (For example, Reserved, 3-year, Partial up front, vs Reserved, 1 year, All up front could be different offers of the same sku).  

[![dashboard][1]][2]

[1]:  https://github.com/pgaljan/blog/assets/11296072/54e107ab-46ea-43e7-ac21-60f318ab5120
[2]:  https://app.powerbi.com/view?r=eyJrIjoiYzRmOTY1MDYtZmE1ZC00MzA5LWFhMjYtMTIzM2Q0MWMwYjBlIiwidCI6ImZlNGQ5NDA3LWE5NzEtNDhjMy1hOTkzLTRjMmNiOGQ2MjM4NCIsImMiOjF9 "Redirect to homepage"

Contractual filters are critical to get refined numbers.  Set as many as can be reasonably determined including:
- Lease Contract Length (On Demand, 1yr, 3 yr)
- Purchase Option (All upfront, no upfront, etc)
- Tenancy (Dedicated, shared, reserved, etc)
- Offer class (convertible, standard)
- Operating System (Windows, Linux, RHEL, etc)
- Pre-Installed SW (SQL Web, Std, Ent)

If you use the dashboard in M365, it should refresh weekly sometime on Sunday.  I pull pricing data for US East, since that region is most likely to have all instance types, and relative prices based on location are relatively easy to derive.  I can look into adding more regions, or replacing US-East with another region.