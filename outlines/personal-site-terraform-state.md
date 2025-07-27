- I configured my personal site to use Terraform with a remote state backend.
    - I set up an GCS bucket to store the Terraform state files.
        - I allowed Service Account Keys
        - Created a Service Account with a associated principal
        - I gave the Service Account principal admin access to a new GCS bucket
    - I added a backend declaration to my Terraform configuration.
        - I specified the GCS bucket as the backend for storing state files.
        - I dynamically prefixed the state file with the environment to allow
            for concurrent deployments.
        - I made sure to call `terraform init -reconfigure` to apply environment
            changes between operations.
