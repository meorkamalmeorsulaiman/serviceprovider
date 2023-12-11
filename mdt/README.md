## XR MDT Configs

### Modes
- Two modes available:
  - Dial-out
  - Dial-in
 
### Dial-out mode steps
- Create destination group
```
Router(config)#telemetry model-driven
Router(config-model-driven)#destination-group <group-name>
Router(config-model-driven-dest)#address family ipv4 <IP-address> port <port-number>  
Router(config-model-driven-dest-addr)#encoding <encoding-format>  
Router(config-model-driven-dest-addr)#protocol <transport> 
Router(config-model-driven-dest-addr)#commit
```

- Create sensor group
```
Router(config)#telemetry model-driven
Router(config-model-driven)#sensor-group <group-name>
Router(config-model-driven-snsr-grp)# sensor-path <XR YANG model>
Router(config-model-driven-snsr-grp)# commit
```
- Create subscription
```
Router(config)#telemetry model-driven  
Router(config-model-driven)#subscription <subscription-name>  
Router(config-model-driven-subs)#sensor-group-id <sensor-group> sample-interval <interval>  
Router(config-model-driven-subs)#destination-id <destination-group>
Router(config-model-driven-subs)#source-interface <source-interface>
Router(config-mdt-subscription)#commit 
```
- Validation
```
show telemetry model-driven subscription <subscription-name>
show telemetry model-driven dest
```

### Dial-in mode steps
Dial-in wheter the destination initate a session to the router - router as a server. The destination box subscribe to the streamed data
- Enable gRPC
```
Router# configure
Router (config)# grpc
Router (config-grpc)# port <port-number> | port range from 57344 to 57999
Router (config)# grpc{ address-family | dscp | max-request-per-user | max-request-total | max-streams | max-streams-per-user | no-tls | service-layer | tls-cipher | tls-mutual | tls-trustpoint | vrf }
```
- Create sensor group
- Create subscription
- Validation
```
show telemetry model-driven subscription <subscription-name>
show telemetry model-driven dest
```
