# Configure DDNS for domene.shop using ubiquiti usg 

Most of the information is written in the [EdgeRouter documentation](https://help.ui.com/hc/en-us/articles/204976324-EdgeRouter-Custom-Dynamic-DNS), but this guide is still missing some important steps to make DDNS work with domeneshop. 

## Setup

First of all, enter the USG through ssh `ssh <IP ADDRESS>`, and enter configuration mode with the command: `configure`

Now you need to configure DDNS for domeneshop.

1. Configure the dynamic DNS hostname
    * Single hostname: 
        
            set service dns dynamic interface eth0 service domeneshop host-name "YOURDOMAIN.com"

    * Multiple hostnames: 
        
            set service dns dynamic interface eth0 service domeneshop host-name "SUBDOMAIN.YOURDOMAIN.com,YOURDOMAIN.com" 

2. Set your domeneshop API credentials, These can be generated through [domeneshop](https://domene.shop/admin?view=api)

        set service dns dynamic interface eth0 service domeneshop login API 'API KEY'
        set service dns dynamic interface eth0 service domeneshop password 'API SECRET'

3. Add domeneshop API uri as the server

        set service dns dynamic interface eth0 service domeneshop server api.domeneshop.no/v0

4. Select dyndns as protocol/template

        set service dns dynamic interface eth0 service domeneshop protocol dyndns

5. Save or discard changes

    * Save:
        
            commit ; save

    * Discard:
        
            discard


## Notes

* Domeneshop's REST API is still under development, and breaking changes may occur 
* Ubiquiti uses ddclient to update DDNS, and the configuration for ddclient is very similar if not exactly the same as the configuration ubiquiti uses. So custom configuration based on ddclient's documentation might work. 
 
## Useful commands

* Show DDNS status

        show dns dynamic status

* Update DNS records

        update dns dynamic interface eth0

* Show DDNS configuration (only works in configuration mode)

        show service dns dynamic

    * Response:
        
            interface eth0 {
                service domeneshop {
                    host-name "SUBDOMAIN.YOURDOMAIN.com,YOURDOMAIN.com"
                    login API-KEY
                    password API-SECRET
                    protocol dyndns
                    server api.domeneshop.no/v0
                }
            }
         
* Remove DDNS configuration (only works in configuration mode)

        delete service dns dynamic
    
        commit ; save

* Show ddclient configuration 
        
        sudo cat /etc/ddclient/ddclient_eth0.conf

    * Response:
        
            server=api.domeneshop.no/v0, protocol=dyndns max-interval=28d login=KEY password='API SECRET' SUBDOMAIN.YOURDOMAIN.com,YOURDOMAIN.com

* Update DNS records through ddclient

        sudo ddclient -file /etc/ddclient/ddclient_eth0.conf -daemon=0 -debug -verbose -noquiet -force


## Refrences

Forum thread about custom DDNS config for USG: 
    
    https://community.ui.com/questions/Add-custom-dynamic-dns-provider-in-usg/2a15d8f0-bef7-422c-bd3a-41a41539d5de 

Custom dyndns configuration guide for EdgeRouters, with cloudflare as dns provider (Ui.com)
    
    https://help.ui.com/hc/en-us/articles/204976324-EdgeRouter-Custom-Dynamic-DNS

Domeneshop API documentation

    https://api.domeneshop.no/docs/#tag/ddns


ddclient GIT repo (executable used to update DDNS)

    https://github.com/ddclient/ddclient
