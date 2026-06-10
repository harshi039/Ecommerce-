import json
import pandas as pd

# Read the file
with open("ret.txt", "r") as f:
    orders = json.load(f)

result = []

for order in orders:

    # Account ID
    account_id = order.get("Cloud", {}).get("AccountId")

    # Metadata
    metadata_str = order.get("Metadata")

    retention_period = None
    retention_unit = "day"

    if metadata_str:
        metadata = json.loads(metadata_str)

        retention_period = metadata.get("retention_period")
        retention_unit = metadata.get("retention_period_unit", "day")

    # Convert to days
    retention_days = None

    if retention_period is not None:

        if str(retention_period).lower() == "permanent":
            retention_days = 99 * 365

        elif retention_unit.lower() == "year":
            retention_days = int(retention_period) * 365

        elif retention_unit.lower() == "month":
            retention_days = int(retention_period) * 30

        else:
            retention_days = int(retention_period)

    result.append({
        "Account ID": account_id,
        "Retention Period": retention_period,
        "Retention Unit": retention_unit,
        "Retention Days": retention_days
    })

# Create Excel
df = pd.DataFrame(result)
df.to_excel("retention_report.xlsx", index=False)

print("Excel file created successfully")
