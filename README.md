# Openshift-Ansible-Automation

These playbooks help to configure OCP after a cluster has been installed.  It is possible to use these playbooks
on many clusters by creating a {repo folder}/vars/{cluster_name} directory to contain the vars.yml and vault.yml 
for each cluster.  

GENERAL:
        These playbooks use the following python3 modules.  They need to be installed prior to running the automation.
           urllib3
           requests
           requests-oauthlib
           openshift (version greater than 0.6)
           PyYAML (version greater than 3.11)
        To install these modules you can use:
           pip3 install <module> --user

	The playbooks use the vars.yml and vault.yml files within a directory under vars matching the name 
        of each openshift cluster you want to administer using these playbooks.

	Do not commit an unencrypted vault.yml file to Github.  The example-vault.yml file 
	is just an example and should not be filled in.

	If a {repo folder}/vars/{cluster_name}/vault.yml doesn't exist use ansible-vault create
        vault.yml to create it.
	If a {repo folder}/vars/{cluster_name}/vault.yml exists use ansible-vault edit vault.yml to edit it.

        The values for the vsphere variables can be found in the install-config.yaml or by doing the following:
          oc get machines <master machine name> -o yaml
        The values are in the spec.workspace section.

USAGE:
	./playbook.yml --ask-vault-pass -t {role name} -e {action}=true -e cluster_name={cluster_name}

	Example:
	Configure ldap authentication:
	./playbook.yml --ask-vault-pass -t ldap -e apply=true -e cluster_name=it-lab

	Teardown ldap authentication:
	./playbook.yml --ask-vault-pass -t ldap -e teardown=true -e cluster_name=it-lab

LDAPs:
	If using ldaps run the following command to get the certs to use in the vault.yml file
		openssl s_client -connect ldaps_connect_URL -showcerts
	#Note: The LDAP role does not create role bindings so in order to access the cluster the RBAC role will need to be run as well.

EGRESS:
	Egress automation allows you to use OVN or CDN networking.
	If SDN:
		Every node will be allowed to host the egress IP. Only 1 IP is allowed per namespace. Each node will be allowed to host the CIDR range provided which should include every IP alloted to a namespace.
		Make sure to review Egress Firewall documentation to understand the structure of the template needed and its limitations.
			For Firewall templates make sure to copy the first 11 lines of the example template provided.
		NOTE: In SDN the teardown will not teardown the egress setup. These values must be removed manually if you wish to remove the egress setup. Remove CIDRs from hostsubnets and EgressIPs from netnamespaces. 
		Egress-Firewall: https://docs.openshift.com/container-platform/4.7/networking/openshift_sdn/configuring-egress-firewall.html
		Egress-IP: https://docs.openshift.com/container-platform/4.7/networking/openshift_sdn/assigning-egress-ips.html
	If OVN:
		Every node will be allowed to host egress IPs. Each namespace can have multiple IP addresses. The CIDR range is not used for OVN.
		Make sure to review Egress Firewall documentation to understand the structure of the template  needed and its limitations.
			For Firewall templates make sure to copy the first 11 lines of the example template provided.
		Egress-Firewall: https://docs.openshift.com/container-platform/4.7/networking/ovn_kubernetes_network_provider/configuring-egress-firewall-ovn.html
		Egress-IP: https://docs.openshift.com/container-platform/4.7/networking/ovn_kubernetes_network_provider/configuring-egress-ips-ovn.html

RBAC:
	This role is used to control OCP rbac role/cluster-role creation as well as role/cluster-role bindings
	To specify a cluster-role:
		- role_name: cluster-role-name
	      group: group-to-bind
		  create_role: true/false      ### (Optional) If true, add a file under roles/rbac/templates with the name 												  cluster-role-name.yml.j2. If false/undefined then the role must already exist within 									 OCP.
	To specify a role:
		- role_name: role-name
		  group: group-to-bind
		  create_role: true/false      ### (Optional) If true, add a file under roles/rbac/templates with the name 												  cluster-role-name.yml.j2. If false/undefined then the role must already exist within 									 OCP.
		  namespaces:         ### Presence of the namespaces list denotes the difference between a role and a cluster-role.
            - namespace1
            - namespace2
	NOTE: Namespaces will be created if they do not exist. The namespaces will not be removed during teardown.

CUSTOM CERTS:
	This role will create secrets that hold certs and their private keys utilizing values in the vault.yml file.
	Each cert that is desired will need a unique key/cert name within the vault.yml file (cert1.crt : | blah, cert2.crt: | blah, cert1.key: | blah, cert2.key: | blah). These can be added utilizing the command ansible-vault edit <repodir>/vars/<cluster>/vault.yml. 
	NOTE: Namespaces will be created if they do not exist. The namespaces will not be removed during teardown.

Manual Approval strategy for Operator installations:
	NOTE: to approve the operator installtion you must choose the project at the top of the installed operators 
	page.  If you choose all namespaces these operators will not be listed because they aren't installed yet.

	If you set the approval strategy to manual in the vars.yml file you must approve the install plan when running
	these roles:

	logging - approve the operator installation in the openshift-logging namespace
          	  approve the operator installation in the openshift-operators-redhat namespace
	compliance - approve the operator installation in the openshift-compliance namespace
	file integrity - approve the operator installation in the openshift-operators namespace

Compliance operator:
	The compliance operator automation does not include a teardown option.  This
	is because when removing the operator the following objects are not removed:
		profilebundles
		compliancescans
		compliancesuites

ETCD Backups:
	If using a proxy ensure you have the shell environment variables exported prior to
	running the etcd_backups automation.  This automation pulls the etcd image from 
	quay.io in order to build the cronjob pods.
	shell environment variables:
		HTTP_PROXY
		HTTPS_PROXY
		NO_PROXY

To run the playbooks:
./playbook.yml --ask-vault-pass -t {role name} -e {action}=true

Example:
Configure ldap authentication:
./playbook.yml --ask-vault-pass -t openshift_config_ldap -e apply=true -e cluster_name=<cluster name>

Teardown ldap authentication:
./playbook.yml --ask-vault-pass -t openshift_config_ldap -e teardown=true -e cluster_name=<cluster name>
