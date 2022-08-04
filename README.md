# CRI_Bigtop_Stack
Configuration to deploy Apache Bigtop on Rocky Linux 8

## Initial OS config: 
1. Enable PowerTools repo
1. Install dependencies:

   ```
      dnf install epel-release vim git strace lsof java ruby puppet \
          wget yum-utils createrepo whois nfs-utils nfs-server bind-utils \
          R-core-devel R-devel openblas-devel httpd python3 python3-devel 
   ```

1. Configure firewall as needed - internal net in trusted zone, public allow 80,443, and 22
1. Generate certificates (this example uses certbot)
1. Ensure https redirect is in place as usual
1. Deploy local bigtop rpm repo

   ```
      cd /etc/yum.repos.d/
      wget https://dlcdn.apache.org/bigtop/bigtop-3.1.0/repos/rockylinux-8/bigtop.repo
      mkdir -p /usr/share/bigtop/releases/bigtop-3.1.0-rockylinux-8/
      reposync --repo bigtop --download-metadata -p /usr/share/bigtop/releases/bigtop-3.1.0-rockylinux-8/
      createrepo /usr/share/bigtop/releases/bigtop-3.1.0-rockylinux-8/
   ```

1. Ensure hostname is resolvable and matches host defined in site.yaml
1. Ensure /opt/bigtop is entered into /etc/exports for NFS-sharing across the cluster

   ```
     /opt/bigtop  10.0.0.0/24(rw,no_subtree_check,no_root_squash)
   ```

1. Edit params in site.yaml appropriately

   (ensure desired modules are included, these can be compared against the
   available modules in
   `/opt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/manifests/cluster.pp`
   )

1. Grab bigtop source release:

   ```
   mkdir -p /opt/bigtop/ && cd /opt/bigtop
   wget https://dlcdn.apache.org/bigtop/bigtop-3.1.0/bigtop-3.1.0-project.tar.gz
   tar xfvz bigtop-3.1.0-project.tar.gz 
   ```

1. Copy hierdata config to main `/etc/puppetlabs/puppet` config dir:
  
   ```
   cp -r /opt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/hieradata/ /etc/puppetlabs/puppet/
   cp -r /opt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/hiera.yaml /etc/puppetlabs/puppet/
   ```

1. DOUBLE CHECK THAT `/etc/puppetlabs/puppet/hiera.yaml` REFLECTS A CORRECT PATH FOR THE PUPPET CONFIG DIR

1. Generate the necessary puppet modules via *gradle*

   ```
   cd /opt/bigtop/bigtop-3.1.0/
   ./gradlew toolchain-puppetmodules
   ```

1. Overwrite the default config with *this* site.yaml file WHICH YOU HAVE ALREADY EDITED, RIGHT?

   ```cp ./site.yaml /etc/puppetlabs/puppet/hieradata/site.yaml```
    
1. Puppet apply! (do this in /tmp to avoid a particular error)
 
   `puppet apply -d --modulepath="/opt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/modules:/etc/puppetlabs/code/modules/" /o
pt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/manifests |& tee puppet_apply.log`

1. Apply httpd proxy config for zeppelin
1. DO NOT OPEN THE FIREWALL TO ANTHING ELSE
1. Change the zeppelin default password - in `/opt/bigtop/bigtop-3.1.0/bigtop-deploy/puppet/modules/zeppelin/manifsts/SOMETHING` OR better yet, do this in your site.yaml and ACL-protect it to within an inch of its life.
1. If you wish to use R within Zeppelin:

   ```ln -s /usr/bin/python3 /usr/bin/python```
   ```pip3 install jupyter-client grpcio protobuf```
   And also install IRKernel via R and CRAN, then somehow convince the jupyter installation to use it correctly.

1. Repeat on other nodes in your cluster as needed! 

   Except don't put *their* name in for the NAMENODE-FQDN. They should be added somewhere in...
   `site.yaml`, I suspect, but we haven't gotten that far yet.
