### deel van de hostvars:
```
"ferm_host_rules": [  
    {  
        "match": "saddr ({% for ssh_host in (groups.jumphost + groups.admin) %}{% if ssh_host != inventory_hostname %}{{ hostvars[ssh_host].ansible_all_ipv4_addresses|join(\" \")}}{% if not loop.last %} {% endif %}{% endif %}{% endfor %})",   
    }  
],   
"ferm_rules_all": [  
    {  
        "match": "saddr ({% for ssh_host in (groups.jumphost + groups.admin) %}{% if ssh_host != inventory_hostname %}{{ hostvars[ssh_host].ansible_all_ipv4_addresses|join(\" \")}}{% if not loop.last %} {% endif %}{% endif %}{% endfor %})",   
    }  
],   
```

### jinja template:
```
{% for group_name in (group_names + ['all']) %}  
{% set group_rules="ferm_rules_" + group_name %}  
{% for rule in (hostvars[inventory_hostname][group_rules]|default([])|safe) %}  
# {{ rule.match }}  
{% do rules.append(rule) %}  
{% endfor %}  
{% endfor %}  
{% for rule in ferm_default_rules %}  
{% for rule in ferm_host_rules|default([]) %}  
{% do rules.append(rule) %}  
# {{ rule.match }}  
{% endfor %}  
```

 
### resultaat:
```
saddr ({% for ssh_host in (groups.jumphost + groups.admin) %}{% if ssh_host != inventory_hostname %}{{ hostvars[ssh_host].ansible_all_ipv4_addresses|join(" ")}}{% if not loop.last %} {% endif %}{% endif %}{% endfor %})  ACCEPT;  
saddr (192.168.56.210 10.0.2.15 10.0.2.15 192.168.56.211)  ACCEPT;  

```
