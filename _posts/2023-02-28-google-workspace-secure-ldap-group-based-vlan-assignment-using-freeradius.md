---
title: "Google Workspace Secure LDAP Group Based VLAN Assignment using FreeRADIUS"
layout: post
---

![screenshot]({{ site.baseurl }}/assets/images/group-based-vlan-assignment/title.png)

In this post we’ll be looking at using FreeRADIUS integrated with Google Workspace Secure LDAP to perform VLAN assignment for WPA enterprise users. This setup has been tested with Ubiquiti Unifi and Cisco Meraki successfully. FreeRADIUS setup and Google Workspace integration has been covered in a previous post so please check it out to understand the prerequisites of this setup.

<!--more-->

To apply policy based on LDAP groups we first need their exact name and syntax. One way of getting that is through FreeRADIUS debug (using *freeradius -X*) as the user is being authenticated. Consider the following partial debug output:

```
(30) ldap: User object found at DN "uid=student,ou=12,ou=Students,ou=Users,dc=example,dc=com"
(30) ldap: Adding cacheable user object memberships
(30) ldap:   &control:LDAP-Group += "cn=2023,ou=Groups,dc=example,dc=com"
(30) ldap:   &control:LDAP-Group += "cn=group4virtual,ou=Groups,dc=example,dc=com"
(30) ldap:   &control:LDAP-Group += "cn=students,ou=Groups,dc=example,dc=com"
```

The last line shows that the user belongs to the *students* group. Suppose we want every user in the *students* group to be assigned to VLAN 10. Here is the process for it.

First create a policy for checking LDAP membership:

```
nano /etc/freeradius/3.0/policy.d/ldap_groupcheck
```

Add the following contents to the file. The RADIUS attribute *Tunnel-Private-Group-ID* specifies the VLAN ID that should be applied to any member of the *students* group.

```
ldap_groupcheck {
	if (&LDAP-Group[*] == "cn=students,ou=Groups,dc=example,dc=com") {
		update reply {
		 &Tunnel-Type := 13
		 &Tunnel-Medium-Type := 6
		 &Tunnel-Private-Group-ID := 10
		}
	}
}
```

Save and exit.

Apply this policy to the *default* virtual-server:

```
nano /etc/freeradius/3.0/sites-enabled/default
```

In *authorize* section after *ldap* add this:

```
ldap_groupcheck
```

In addition to this, right at the start of *authorize* section, add this directive:

```
preprocess
```

Save and exit.

Update the *eap* module:

```
nano /etc/freeradius/3.0/mods-enabled/eap
```

In *ttls* section:

```
copy_request_to_tunnel = yes
```

Save and exit.

Finally, update the *ldap* module:

```
nano /etc/freeradius/3.0/mods-enabled/ldap
```

In *group* section:

```
cacheable_dn = 'yes'
```

Save and exit.

Restart FreeRADIUS service for new settings to take effect:

```
systemctl restart freeradius.service
```

### Rules for Multiple Groups

The unlang code for *ldap_groupcheck* given above can be extended to as many groups and VLANs as we like. For example, for 3 different groups of *students*, *teachers* and *admin* that need to be assigned to VLANs 10, 20 and 30 respectively, we can use the following code:

```
ldap_groupcheck {
	if (&LDAP-Group[*] == "cn=students,ou=Groups,dc=example,dc=com") {
		update reply {
			&Tunnel-Type := 13
			&Tunnel-Medium-Type := 6
			&Tunnel-Private-Group-ID := 10
		}
	}
	elsif (&LDAP-Group[*] == "cn=teachers,ou=Groups,dc=example,dc=com") {
		update reply {
			&Tunnel-Type := 13
			&Tunnel-Medium-Type := 6
			&Tunnel-Private-Group-ID := 20
		}
	}
	elsif (&LDAP-Group[*] == "cn=admin,ou=Groups,dc=example,dc=com") {
		update reply {
			&Tunnel-Type := 13
			&Tunnel-Medium-Type := 6
			&Tunnel-Private-Group-ID := 30
		}
	}
}
```

Similarly, if a user is part of multiple groups, like *teachers* and *admin*, and we want to assign them a particular VLAN, we can create a rule for such cases as well:

```
ldap_groupcheck {
	if (&LDAP-Group[*] == "cn=teachers,ou=Groups,dc=example,dc=com" && &LDAP-Group[*] == "cn=admin,ou=Groups,dc=example,dc=com") {
		update reply {
			&Tunnel-Type := 13
			&Tunnel-Medium-Type := 6
			&Tunnel-Private-Group-ID := 20
		} 
	}
}
```

Note that the rules are parsed from top to bottom and if a user matches a certain rule then the proceeding rules are ignored.