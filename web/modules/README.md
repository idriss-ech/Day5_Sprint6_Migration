<h1 style="font-size:80px;color:#ef233c;text-align:center;">
DAY 5
<br>

<span style="color:#2b2d42;"> Work with Data Migration</span>
![image](https://github.com/user-attachments/assets/b946aff9-f14a-471f-b58c-cfe69835f0d2)

</h1>

<h1 style="color:#8d99ae;">Introduction</h1>
<p style="text-align:justify;">
Drupal provides a powerful Migration API that allows you to import data from various sources into Drupal entities (nodes, taxonomy, users, etc.). The migration process is based on <strong>ETL</strong> (Extract, Transform, Load) principles. 
<br>

Drupal's migration system revolves around three key components:

1. **Source:** The data source (e.g., CSV, JSON, XML, SQL database, etc.).
2. **Process:** The transformations applied to the data before importing.
3. **Destination:** The final entity type in Drupal (e.g., nodes, users, taxonomy terms).
   Drupal migrations are defined in YAML files and can be executed using Drush or the UI.

<h1 style="color:#8d99ae;">Example Migration of Articles from a CSV Data Source</h1>

### **Step 1: Install Required Modules**

1. Install the necessary modules using Composer:
   ```bash
    composer require 'drupal/migrate_tools:^6.0'
    composer require 'drupal/migrate_plus:^6.0'
    composer require 'drupal/migrate_source_csv:^3.7'
   ```
2. Enable the modules

---

### **Step 2: Prepare the CSV File**

1. Create a CSV file (`nodes.csv`) with the following structure:
   ```csv
   id,title,body
   1,"First Post","This is the body of the first post."
   2,"Second Post","This is the body of the second post."
   3,"Third Post","This is the body of the third post."
   4,"Fourth Post","This is the body of the fourth post."
   5,"Fifth Post","This is the body of the fifth post."
   6,"Sixth Post","This is the body of the sixth post."
   7,"Seventh Post","This is the body of the seventh post."
   8,"Eighth Post","This is the body of the eighth post."
   9,"Ninth Post","This is the body of the ninth post."
   10,"Tenth Post","This is the body of the tenth post."
   ```
2. Place the file in the `public://import-sources` directory (e.g., `myproject/web/sites/default/files/import-sources/nodes.csv`).

---

### **Step 3: Create a Custom Module**

1. Create a custom module named `custom_migration`:
2. Inside the module, we create the following files:

   - `custom_migration.info.yml`:
<br>

   ```yaml
   name: "Custom Migration"
   type: module
   description: "Handles custom CSV imports."
   core_version_requirement: ^9 || ^10
   package: "Custom"
   dependencies:
     - drupal:migrate
     - drupal:migrate_plus
     - drupal:migrate_tools
     - drupal:migrate_source_csv
   ```
   - `config/install/migrate_plus.migration.custom_csv_import.yml`:
<br>
    ```yaml

      id: custom_csv_import
      label: Import articles
      migration_group: default
      source:
        plugin: "csv"
        path: "public://import-sources/nodes.csv"
        delimiter: ","
        enclosure: '"'
        header_offset: 0
        ids:
          - id
        fields:
          - name: id
            label: "Unique Id"
          - name: title
            label: "Title"
          - name: body
            label: "Body"
      process:
        title: title
        body: body
        type:
          plugin: default_value
          default_value: article
      destination:
        plugin: entity:node
     ```

---

### **Step 4: Enable the Custom Module**

1. Enable the custom module:
   ```bash
   drush en custom_migration
   ```

---

### **Step 5: Run the Migration**

1. Run the migration to import the CSV data:
   ```bash
   drush migrate:import custom_csv_import
   ```
   <img width="871" alt="image" src="https://github.com/user-attachments/assets/97ac49e6-5aeb-4f7f-81ec-810029837a9c" />


---

### **Step 6: Verify the Imported Content**

<img width="1439" alt="image" src="https://github.com/user-attachments/assets/30002cf3-f6aa-4b1e-8b59-4f949e9c9ca5" />

---


</p>

<h1 style="color:#ff4d6d;">I. What's the role of <span style="color:#2b2d42;"> migration_lookup</span> </h1>
migration_lookup` is a Drupal process plugin that lets us reference previously migrated entities. It takes IDs from our source data and converts them to the corresponding Drupal entity IDs, making it essential for maintaining relationships between content during migration (like connecting nodes to authors or taxonomy terms).
Example: 
```yaml
process:
  field_category:
    plugin: migration_lookup
    migration: taxonomy_term_categories
    source: category_id
```
migration: Specifies which migration(s) to check for the entity
source: The source field containing the ID to look up

In this example, field_category is populated by looking up the destination ID (Drupal entity ID) that corresponds to the source category_id from a previous migration called taxonomy_term_categories.

<h1 style="color:#ff4d6d;">II. What would you do if you needed to import from a different data source other than CSV, say MySQL database ? </h1>
If we needed to import from a MySQL database instead of CSV in Drupal, we would:

1. Use the `migrate_source_sql` module, which provides database source plugins
2. Define our migration source using the `d7_database` or `sql` source plugin
3. Configure the database connection in settings.php or directly in the migration definition
4. Write SQL queries in our migration YAML to extract the data

<b>Example of the source section :</b>

```yaml
source:
  plugin: sql
  query: |
    SELECT id, title, body, created 
    FROM source_table
    WHERE status = 1
  keys:
    - id
  database_connection_key: migrate
```

<h1 style="color:#ff4d6d;">II. How would you rollback a migration ?</h1>

To rollback a migration in Drupal, we would use the drush command line tool with the following command:

```bash
drush migrate:rollback [migration_id]
```

Where <b>[migration_id]</b> is the machine name of the migration we want to rollback.

**Options**

```bash
--all. Process all migrations
--tag=TAG. A comma-separated list of migration tags to rollback
--feedback=FEEDBACK. Frequency of progress messages, in items processed
--idlist=IDLIST. Comma-separated list of IDs to rollback. As an ID may have more than one column, concatenate the columns with the colon ':' separator
--progress[=PROGRESS]. Show progress bar [default: 1]
--no-progress. Negate --progress option.
```

<h1 style="color:#ff4d6d;">IV. How would you process a field data source before importing it ? Say trim the length of a string to 10 characters ?</h1>
we would use the process plugins provided by Drupal's Migration API.

Example :

```php
process:
  destination_field:
    -
      plugin: callback
      source: source_field
      callable: substr
      callable_arguments:
        - 0  // Starting position (0-based index)
        - 10 // Maximum length
```

- The callback plugin exposes PHP functions to our migration pipeline
- substr() is a native PHP string manipulation function
- The arguments array passes parameters to the function in order (start position, length)
