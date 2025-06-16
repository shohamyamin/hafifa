# 🧪 Assignment: Trino + Iceberg + Superset Stack via Docker Compose

### 🎯 Goal

Build a complete analytics stack using **Docker Compose** that includes:

* **Trino** for SQL-based data querying
* **Iceberg** with a REST catalog backed by MinIO for storage
* **Superset** for building visual dashboards

You will:

1. Set up each service using Docker Compose
2. Configure Trino to use TPCH and Iceberg catalogs
3. Populate Iceberg with data from TPCH
4. Connect Superset to Trino
5. Build a basic dashboard in Superset based on Iceberg tables

---

## 📁 Step 1: Set Up Project Structure

Create a working directory to contain all files:

```bash
mkdir docker-trino-iceberg-stack
cd docker-trino-iceberg-stack
mkdir -p trino/catalog
```

You will place your Trino catalog configuration files under `trino/catalog`.

---

**❓ Question 1:**
What is the role of each component (Trino, Iceberg, MinIO, Superset) in this data stack?

---

## ⚙️ Step 2: Write the Docker Compose Configuration

Create a `docker-compose.yml` file that defines the following services:

1. **MinIO**

   * Acts as the object store used by Iceberg
   * Should be accessible on ports 9000 (API) and 9001 (console)
   * Needs environment variables for root credentials

2. **Iceberg REST Catalog**

   * Uses the `tabulario/iceberg-rest` image
   * Connects to the MinIO service
   * Needs environment variables for warehouse path, credentials, and S3 endpoint

3. **Trino**

   * Uses the official `trinodb/trino` image
   * Mounts a folder with catalog configuration files
   * Should expose port 8080
   * Needs to depend on both MinIO and Iceberg REST Catalog services

4. **Superset**

   * Uses the official `apache/superset` image
   * Exposes port 8088
   * Needs environment variables to initialize an admin user

Make sure to include a **shared Docker volume** for MinIO storage.

---

**❓ Question 2:**
Why does Iceberg require both an object store like MinIO and a catalog service?

---

## 📚 Step 3: Configure Trino Catalogs

Inside the `trino/catalog` folder, create two files:

1. `tpch.properties` – defines the TPCH connector

```ini
connector.name=tpch
```

2. `iceberg.properties` – defines the Iceberg REST catalog

This file should:

* Use `connector.name=iceberg`
* Set catalog type to `rest`
* Provide the REST catalog URI (the iceberg-rest container)
* Set warehouse to `s3a://warehouse`
* Include S3 settings like endpoint URL and access credentials

---

**❓ Question 3:**
What’s the difference between using Iceberg with Hive Metastore vs. the REST catalog?

---

## 🧪 Step 4: Launch the Stack

Use the following command to start all services:

```bash
docker-compose up -d
```

Once running, open a shell into the Trino container:

```bash
docker exec -it trino trino
```

Run a test:

```sql
SHOW CATALOGS;
```

You should see `tpch` and `iceberg`.

---

**❓ Question 4:**
Why do we need both catalogs (`tpch` and `iceberg`) in Trino for this assignment?

---

## 🧱 Step 5: Populate Iceberg Tables from TPCH

In the Trino CLI:

1. Create a schema in Iceberg:

   ```sql
   CREATE SCHEMA IF NOT EXISTS iceberg.tpch;
   ```

2. For each table in `tpch.tiny` (e.g., `nation`, `customer`, `orders`, etc.), run:

   ```sql
   CREATE OR REPLACE TABLE iceberg.tpch.<table_name> AS SELECT * FROM tpch.tiny.<table_name>;
   ```

   You can repeat this for all 8 TPCH tables:

   * customer
   * lineitem
   * nation
   * orders
   * partsupp
   * part
   * region
   * supplier

---

**❓ Question 5:**
What’s the difference between `CREATE TABLE AS SELECT` and `INSERT INTO` in Trino?

---

## 📊 Step 6: Configure Superset

1. Access Superset at: [http://localhost:8088](http://localhost:8088)

2. Login with the admin credentials you set in the `docker-compose.yml`

3. Add a **new database connection** to Trino:

   * SQLAlchemy URI: `trino://user@trino:8080`
   * No password is required for the basic setup

4. Test the connection and save

---

**❓ Question 6:**
Why is Superset useful when working with SQL engines like Trino?

---

## 📈 Step 7: Build a Dashboard in Superset

Create a dashboard that includes:

1. A **bar chart** showing total orders per customer from `iceberg.tpch.orders`
2. A **table** showing basic customer info from `iceberg.tpch.customer`

Steps:

* Create datasets from the Trino catalog → iceberg.tpch tables
* Use the Superset visualization interface to create the charts
* Save them into a new dashboard

---

**❓ Question 7:**
What are some benefits of visualizing data using dashboards instead of raw SQL?

---

## ✅ Completion Checklist

* [ ] Docker Compose file written and all services launched
* [ ] Trino catalogs `tpch` and `iceberg` are correctly configured
* [ ] All TPCH tables copied to Iceberg using `CREATE TABLE AS SELECT`
* [ ] Superset is running and connected to Trino
* [ ] A dashboard is created in Superset using Iceberg data
