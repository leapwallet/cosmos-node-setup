# To-dos

- Use EC2 Spot instances, savings plans, and budget alerts. Use two EC2 instances in different AZs for redundancy with a load balancer. Obviously, only back up one of the AWS EBS volumes.
- Use Fish instead of the default shell.
- Use Terraform.
- Set up rate limiting.
- Set up the DNS.
- Set up Prometheus. Doc that they'll have to add a security group for ports 26660 (Cosmos) and 9100 (node_exporter) over TCP exposed to Prometheus if they want to use Prometheus outside the EC2 instance. Doc exposing the Prometheus port 
- Set up Grafana (e.g., create dashboard, set up alerts, reports).
- Set up PANIC.
- Set up logs.
- Skipping the upgradation process by simply downloading the archive.
- Use Docker.
- Automate installation similar to how Osmosis did.
- Things such as firewalls and system requirements can be taken out of the AWS section.
