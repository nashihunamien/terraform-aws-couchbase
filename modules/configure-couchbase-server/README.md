# Couchbase Server Run Script

This folder contains a script for configuring and initializing Couchbase on an [AWS](https://aws.amazon.com/) server. 
This script has been tested on the following operating systems:

* Ubuntu 16.04
* Amazon Linux

There is a good chance it will work on other flavors of Debian, CentOS, and RHEL as well.




## Quick start

This script assumes you installed it, plus all of its dependencies (including Couchbase itself), using the 
[install-couchbase-server module](https://github.com/gruntwork-io/terraform-aws-couchbase/tree/master/modules/install-couchbase-server). 
The default install path is `/opt/couchbase/bin`, so to configure and start Couchbase, you run:

```
/opt/couchbase/bin/configure-couchbase-server --cluster-username <USERNAME> --cluser-password <PASSWORD>
```

This will:

1. Figure out a rally point for your Couchbase cluster. This is a "leader" node that will be responsible for 
   initializing the cluster and/or replication.
   
1. Initialize the cluster, including configuring which services to run, credentials, and memory settings.

1. Add nodes to the cluster.

1. Rebalance the cluster.
   
We recommend using the `configure-couchbase-server` command as part of [User 
Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-shell-scripts), so that it executes
when the EC2 Instance is first booting. 

See the [examples folder](https://github.com/gruntwork-io/terraform-aws-couchbase/tree/master/examples) for 
fully-working sample code.




## Command line Arguments

Run `configure-couchbase-server --help` to see all available arguments.

```
Usage: configure-couchbase-server [options]

This script can be used to configure and run a Couchbase Server. This script has been tested with Ubuntu 16.04 and Amazon Linux.

Required arguments:

  --cluster-username		  The username for the Couchbase cluster.
  --cluster-password		  The password for the Couchbase cluster.

Optional arguments:

  --services			          Comma-separated list of Couchbase service to run. Default: data,index,query,fts.
  --cluster-name		        The name of the Couchbase cluster. Default: use the name of the Auto Scaling Group.
  --cluster-port		        The port the Couchbase cluster will use. Default: 8091.
  --index-storage-setting   The index storage mode for the index service. Must be one of: default, memopt. Default: default.
  --hostname			          The hostname to use for this node. Default: look up the node's private hostname in EC2 metadata.
  --use-public-hostname		  If this flag is set, use the node's public hostname from EC2 metadata.
  --rally-point-hostname	  The hostname of the rally point server that initialized the cluster. If not set, automatically pick a rally point server in the ASG.
  --manage-memory-manually  If this flag is set, you can set memory settings manually via the --data-ramsize, --fts-ramsize, and --index-ramsize arguments.
  --data-ramsize		        The data service memory quota in MB. Only used if --manage-memory-manually is set.
  --index-ramsize		        The index service memory quota in MB. Only used if --manage-memory-manually is set.
  --fts-ramsize			        The full-text service memory quota in MB. Only used if --manage-memory-manually is set.
  --help			              Show this help text and exit.

Example:

  configure-couchbase-server --cluster-username admin --cluser-password password
```



## Passing credentials securely

The `configure-couchbase-server` requires that you pass in your cluster username and password. You should make sure to never 
store these credentials in plaintext! You should use a secrets management tool to store the credentials in an encrypted
format and only decrypt them, in memory, just before calling `configure-couchbase-server`. Here are some tools to consider:

* [Vault](https://www.vaultproject.io/)
* [Keywhiz](https://square.github.io/keywhiz/)
* [KMS](https://aws.amazon.com/kms/)

Moreover, if you're ever calling `configure-couchbase-server` interactively (i.e., you're manually running CLI commands
rather than executing a script), be careful of passing credentials directly on the command line, or they will be 
stored, in plaintext, [in Bash 
history](https://www.digitalocean.com/community/tutorials/how-to-use-bash-history-commands-and-expansions-on-a-linux-vps)!
You can either use a CLI tool to set the credentials as environment variables or you can [temporarily disable Bash
history](https://linuxconfig.org/how-to-disable-bash-shell-commands-history-on-linux). 




## Required permissions

The `configure-couchbase-server` script assumes it is running on an EC2 Instance with an [IAM 
Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) that has the following permissions:

* `ec2:DescribeInstances`
* `ec2:DescribeTags`
* `autoscaling:DescribeAutoScalingGroups`

These permissions are automatically added by the [couchbase-cluster 
module](https://github.com/gruntwork-io/terraform-aws-couchbase/tree/master/modules/couchbase-cluster).
