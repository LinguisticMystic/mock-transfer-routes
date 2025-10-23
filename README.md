# Import and Synchronize Route Data from Mock APIs

## User Story
I need to import and synchronize transfer route data from a mock API so that our internal database always contains the latest information about available routes and prices.

## Description
Build a **console application (Yii console command)** that:

1. Imports transfer route data from a JSON file (initial dataset) into a local database.  
2. Reads a second JSON file (updated dataset) simulating an API update and updates existing data accordingly.  
3. Detects and logs:
   - **Added routes** (new entries)  
   - **Updated routes** (name or price changed)  
   - **Removed routes** (missing from the update)  
4. Converts all prices to EUR (conversion rate: **1 CHF = 1.08 EUR**).  
5. Outputs a summary report to the console and JSON file.

## Mock Data
- **Initial routes:**  
  [https://raw.githubusercontent.com/LinguisticMystic/mock-transfer-routes/refs/heads/main/data/initial_routes.json](https://raw.githubusercontent.com/LinguisticMystic/mock-transfer-routes/refs/heads/main/data/initial_routes.json)  
- **Updated routes:**  
  [https://raw.githubusercontent.com/LinguisticMystic/mock-transfer-routes/refs/heads/main/data/updated_routes.json](https://raw.githubusercontent.com/LinguisticMystic/mock-transfer-routes/refs/heads/main/data/updated_routes.json)  

## Steps

1. Create a console command to handle both import and sync:
   ```
   php yii routes/sync <routes_json_file>
   ```

3. Create a database table with the following columns:

   - `id`
   - `from_location`
   - `to_location`
   - `price` – original price from JSON
   - `currency` – original currency code
   - `price_eur` – converted EUR price
   - `active` – mark removed routes as inactive
   - `created_at`
   - `updated_at`

4. Parse the provided JSON file.

5. For each record:

   - If a route **exists in the DB by ID** → update it if the name or price has changed.  
   - If it **does not exist** → insert it as a new route.

6. Detect **removed routes** (IDs in DB but not in the JSON) → mark them as inactive.

7. Convert CHF → EUR (use **1 CHF = 1.08 EUR**) when storing `price_eur`.

8. Output a simple summary report to the console showing how many routes were added, updated, or removed, and save the report as a JSON file.

## Expected Console Output

```
$ php yii routes/sync data/initial_routes.json
Sync started...
Sync completed.
Summary: 7 added.

$ php yii routes/sync data/updated_routes.json
Sync started...
Sync completed.
Summary: 2 added, 3 updated, 2 removed.
```

## Expected JSON File

```
{
  "added": [108, 109],
  "updated": [101, 102, 105],
  "removed": [104, 106]
}
```
