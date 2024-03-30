### ETL System using Event-Driven Choreography

This ETL (Extract, Transform, Load) system is designed around the event-driven choreography pattern, leveraging the benefits of AWS serverless components. The event-driven choreography pattern allows each service to perform a specific task in response to events, ensuring independence and flexibility.

#### Solution Overview:
1. **External Upload**:
   - An external client uploads files to an S3 bucket folder, triggering the `dataharvest_persistence` lambda.
   - The `dataharvest_persistence` lambda downloads the file, connects to PostgreSQL, and saves data to the DB. It then creates a row in DynamoDB, acting as a metastore to track file processing.

2. **Relationship Building**:
   - DynamoDB stream triggers `dataharvest_relationship_transformer` to establish relationships among PostgreSQL tables.
   - Additional transformers are triggered by a different DynamoDB stream:
     - `dataharvest_sum_deposit_transformer`: Calculates sum deposits.
     - `dataharvest_transaction_transformer`: Calculates transactions (unfinished).
     - `dataharvest_tax_transformer`: Calculates tax for clients (unfinished).

3. **Flexibility and Adaptability**:
   - The system is designed to adapt to changing requirements. New transformers can be accommodated by updating the metastore and changing filter criteria for the `dataharvest_catapult` lambda.
   
#### Solution Details:
- **Assumptions**:
  - Mimicked AWS resource access due to lack of personal AWS account.
  - Small CSV file row count handled by a single lambda instance.
  - No batching considered in the first solution due to concurrency complexity.
  - Assumed a single external client, but code allows for multiple external clients.

- **Sample DynamoDB Metadata**:
  ```json
  {
    "partition_key": "EXT_CLIENT",
    "sort_key": "20220226",
    "account_persisted": true,
    "portfolio_persisted": true,
    "transaction_persisted": true,
    "client_persisted": true
  }
  ```

- **DataHarvest and Architecture**:
  - The system is built on the event-driven choreography pattern using AWS serverless technologies.
  - The choreography pattern distributes tasks across services to prevent a single point of failure and allows for easy updates and replacements.
  
- **Security Measures**:
  - A security mechanism at the system edge includes RBAC-based authorization and various authentication mechanisms like SSO, username/password.
  - Communication is secured with TLS-based encryption, with certificates rotated every 90 days.
  - Sensitive data in the database is encrypted, and regular audits are conducted to detect malicious activities.

- **Regulatory Compliance**:
  - Compliance with GDPR and ISO27001 involves regular backups and restores, vulnerability scans on code, network layer security like WAF, firewall, and network whitelisting.
  
- **Scalability**:
  - DataHarvest maintains a metastore and uses DynamoDB stream batching for scalability.
  - System can scale concurrently using batching of DynamoDB stream and Lambda functions.
  - New transformers can be accommodated by updating the metastore and changing filter criteria for the `dataharvest_catapult` lambda.
   
#### Conclusion:
This solution is a starting point to showcase the benefits of event-driven architecture in ETL systems. Please note that some components like `dataharvest_transaction_transformer` and `dataharvest_tax_transformer` are marked as unfinished due to incomplete implementation.


