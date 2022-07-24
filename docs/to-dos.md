# To-dos

- Optimize AWS costs via savings plans.
- Use Fish instead of the default shell.
- Use Terraform.
- Set up rate limiting.
- Set up the DNS.
- Set up Prometheus.
- Set up Grafana.
- Set up PANIC.
- Set up logs.
- Skipping the upgradation process by simply downloading the archive. Maybe Docker can be used to more easily set up the
  server then.
- Only expose the Prometheus server to the Grafana one in the firewall.
- See if you need to set the halt height in the service file. Also, maybe "ExecStart" can be shortened
  from `/home/ubuntu/go/bin/junod` to `junod`. Also, maybe we can set `Restart=always` in the service file always.
- Document enabling APIs.
- Ensure that the "###Upgrading" section's point 8. iii. has the correct halt height (maybe it's supposed to be -1 by
  default, or something like that). Also, check if after step 9, something like `sudo systemctl restart junod` must be
  run.
