# CRI_Bigtop_Stack
Configuration to deploy Apache Bigtop

1. Initial OS config: 
1. Install dependencies
1. Configure firewall as needed - internal net in trusted zone, public allow 80,443, and 22
1. Generate certificates (this example uses certbot)
1. Ensure https redirect is in place as usual
1. Deploy local bigtop rpm repo
1. Ensure hostname is resolvable and matches host defined in site.yaml
1. Edit params in site.yaml appropriately

   (ensure desired modules are included, these can be compared against the
   available modules in
   `/opt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/manifests/cluster.pp`
   )

1. Puppet apply! 
 
   `puppet apply -d --modulepath="/opt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/modules:/etc/puppetlabs/code/modules/" /o
pt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/manifests |& tee puppet_apply.log`

1. Apply httpd proxy config for zeppelin
1. DO NOT OPEN THE FIREWALL TO ANTHING ELSE
