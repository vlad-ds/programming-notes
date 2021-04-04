# CLOUD SQL

To create an SQL database on GCP: 

1. Sign up for GCP. 
2. Create a project.
3. Start the SQL service.
4. Create database.
5. Create user.

## Cloud SQL Auth Proxy

https://cloud.google.com/sql/docs/mysql/sql-proxy

Cloud SQL Auth proxy is the preferred way of connecting to the DB. It provides secure access to your instances without the need for authorized networks. It removes the need to provide static IP addresses.

To use the Cloud SQL Auth proxy, you must meet the following requirements:

- The Cloud SQL Admin API must be enabled.

- You must provide the Cloud SQL Auth proxy with [Google Cloud authentication credentials](https://cloud.google.com/sql/docs/mysql/connect-admin-proxy#authentication-options).

- You must provide the Cloud SQL Auth proxy with a valid database user account and password.

- The instance must either have a public IPv4 address, or be configured to use [private IP](https://cloud.google.com/sql/docs/mysql/private-ip).

  The public IP address does not need to be accessible to any external address (it does not need to be added as an authorized network address).

To create a service account: 

1. Go to the Service accounts page of the Google Cloud Console.
2. Select the project that contains your Cloud SQL instance.
3. Click **Create service account**.
4. In the **Create service account** dialog, enter a descriptive name for the service account.
5. Change the **Service account ID** to a unique, recognizable value and then click **Create**.
6. For Role, select one of the following roles:
   - **Cloud SQL > Cloud SQL Client**
   - **Cloud SQL > Cloud SQL Editor**
   - **Cloud SQL > Cloud SQL Admin**

7. Click the action menu for your new service account and then select **Manage keys**.
8. Click the **Add key** drop-down menu and then click **Create new key**.
9. Confirm that the key type is JSON and then click Create.
10. The private key file is downloaded to your machine. You can move it to another location. Keep the key file secure.





