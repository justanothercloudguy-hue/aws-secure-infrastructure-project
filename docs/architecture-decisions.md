Decision: SSM Session Manager over SSH
When setting up EC2 access, I initially started with an SSH bastion host. AWS recommended SSM Session Manager instead.
SSM eliminates the need to create and manage SSH keys, which reduces the risk of keys being lost or exposed. It also means no inbound port 22 needs to be open on the EC2 instance, which removes an attack surface. Access is controlled through IAM roles and policies instead.
