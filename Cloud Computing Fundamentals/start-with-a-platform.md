# Start with platform

### Google Cloud console
- Simple web based GUI
- Easily find resources, check health, have full management control and set budgets
- Provide ssh access to resources

### Orgainzation node -> Folders -> Projects -> Resources

### Projects
- Each project is a compartment
- Each resource belongs to a project
- Can have different users and owners
- Each project has projectID(unique), project name and project number(unique)

### Folders
- Folders allow to group resources on per-department basis

### Billing
- Established at project level
- Can be linked to zero or more projects
- Alerts and budgets can be set by the users
- Two types of quotas: Rate quota and Allocation quota 
- Rate quota: Resets after a specific time
- Allocation quota: Governs the number of resources in a project

### CloudSDK
- Set of tools to manage resources hosted on GC
- Consists of gcloud CLI, gsutil, bq
- gcloud CLI: Provides main cmd line interface
- gsutil: Access to Cloud Storage via cmd
- bq: cmd line tool for Big Query 

