Wazuh Docker Installation Instructions

Wazuh Stack Deployment on Server
1) Increase max_map_count on your host (Linux). This command must be run with root permissions:

    sysctl -w vm.max_map_count=262144

2) Start the environment with docker compose from the "~/wazuh-docker/wazuh-image-tars/" folder:

    docker load -i wazuh-manager_4.14.2.tar    
    docker load -i wazuh-indexer_4.14.2.tar
    docker load -i wazuh-dashboard_4.14.2.tar
    docker load -i wazuh-certs-generator.tar


Windows XP Wazuh Installation Instructions and File Location "~/Wazuh/wazuh-agent"
1.) Place wazuh-agent-4.14.2-1.msi onto Windows XP Host

2.) In Command Prompt run, "wazuh-agent-4.14.2-1.msi /q WAZUH_MANAGER="X.X.X.X""
    Once ran, Authentication Key should populate

3.) NET START WazuhSvc


XP Problem Points-
1.) Problem: The XP Machines are not logging all possible log combinations leading to missing data. 
Fix Action:    Run -> secpol.msc -> Local Policies -> Audit Policies -> (Please enable Success and Failure for all Policies as shown below).

2.) Windows XP Logs have a maximum log size for Application, Security, and System. This is leading to missing logs once these stores fill.
Fix Action:    Run -> eventvwr.msc -> Application/Security/System -> Properties -> Enable "Overwrite events as needed"


External Port Mapping:

Port : Component
1514 : Wazuh TCP
1515 : Wazuh TCP
514 : Wazuh UDP
55000 : Wazuh server API
9250 : Wazuh indexer API (Originally mapped to 9200, changed to avoid collision with Elastic)
5602 : Wazuh dashboard HTTPS
