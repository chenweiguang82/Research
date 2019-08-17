# 安装MongoDB数据库 #
由于安装atomate需要安装这个数据库，所以只能突击下这个了。  
## Archlinux下安装 ##
Archlinux的[AUR](https://aur.archlinux.org)功能太强大了，直接安装即可。
```
yay -S mongodb-bin
```
## 启动MongoDB数据库 ##
```
systemctl start mongodb.service
systemctl enable mongodb.service
```
## 配置MongoDB用户 ##
```
(atomate) [zznu@archlinux atomate]$ mongo                                                  
MongoDB shell version v4.0.12                                                                                                                                                          
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb                        
Implicit session: session { "id" : UUID("4494979a-a61c-4323-8b17-7d015e5410e4") }          
MongoDB server version: 4.0.12                                                             
Server has startup warnings: 
2019-08-17T03:24:24.533+0800 I STORAGE  [initandlisten]                                    
2019-08-17T03:24:24.533+0800 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine                                
2019-08-17T03:24:24.533+0800 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem                                                            
2019-08-17T03:24:25.937+0800 I CONTROL  [initandlisten]                                                                                                                                
2019-08-17T03:24:25.938+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.                                                                    
2019-08-17T03:24:25.938+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.                                                   
2019-08-17T03:24:25.938+0800 I CONTROL  [initandlisten]                                    
2019-08-17T03:24:25.939+0800 I CONTROL  [initandlisten]                                    
2019-08-17T03:24:25.939+0800 I CONTROL  [initandlisten] ** WARNING: You are running on a NUMA machine.                                                                                 
2019-08-17T03:24:25.939+0800 I CONTROL  [initandlisten] **          We suggest launching mongod like this to avoid performance problems:
2019-08-17T03:24:25.939+0800 I CONTROL  [initandlisten] **              numactl --interleave=all mongod [other options]
2019-08-17T03:24:25.939+0800 I CONTROL  [initandlisten] **          We suggest launching mongod like this to avoid performance problems:                                      [62/1819]
2019-08-17T03:24:25.939+0800 I CONTROL  [initandlisten] **              numactl --interleave=all mongod [other options]                                                                
2019-08-17T03:24:25.939+0800 I CONTROL  [initandlisten]                  
---                                                                                        
Enable MongoDB's free cloud-based monitoring service, which will then receive and display                                                                                              
metrics about your deployment (disk utilization, CPU, operation statistics, etc).
                                                                                                                                                                                       
The monitoring data will be available on a MongoDB website with a unique URL accessible to you                                                                                         
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.
                                                                                                                                                                                       
To enable free monitoring, run the following command: db.enableFreeMonitoring()     
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()                                                                                            
---                       
> show dbs                                                                                                                                                                             
admin   0.000GB                                                                            
config  0.000GB                                                                                                                                                                        
local   0.000GB                                                                            
> db                                                                                       
test                                                                                       
```
**切换到admin数据库，开始创建用户**
```
> use admin                  
switched to db admin                                                                       
> db                                                                                                                                                                                   
admin
```
**创建root权限用户**
```
> db.createUser({user:"root",pwd:"root",roles:["root"]})                                                                                                                               
Successfully added user: { "user" : "root", "roles" : [ "root" ] }                                                                                                                     
> show dbs                                                                                                                                                                             
admin   0.000GB                                                                            
config  0.000GB                                                                            
local   0.000GB                                                                                                                                                                        
```
**创建admin权限用户**
```
> db.createUser({user:"admin",pwd:"admin",roles:[{role: "userAdminAnyDatabase", db: "admin"}]})                                         
Successfully added user: {
        "user" : "admin",                                                                  
        "roles" : [                                                                        
                {                                                                                                                                                                      
                        "role" : "userAdminAnyDatabase",                         
                        "db" : "admin"                                                                                                                                                 
                }                                                                                                                                                                      
        ]                                                                                  
}                                                                                          
> exit                                                                                                                                                                                 
bye
```
**编辑配置文件，开启安全认证**
```
(atomate) [zznu@archlinux atomate]$ sudo vi /etc/mongodb.conf                                                                                                                          
[sudo] password for zznu: 
```
```
security:
  authorization: enabled//启用授权
```
**重启数据库**
```
(atomate) [zznu@archlinux atomate]$ sudo systemctl restart mongodb.service                 
(atomate) [zznu@archlinux atomate]$ sudo systemctl status mongodb.service                                                                                                              
* mongodb.service - High-performance, schema-free document-oriented database               
   Loaded: loaded (/usr/lib/systemd/system/mongodb.service; enabled; vendor preset: disabled)                                                                                          
   Active: active (running) since Sun 2019-08-18 01:22:06 HKT; 18s ago                     
 Main PID: 10463 (mongod)                                                                  
    Tasks: 27 (limit: 4915)                                                                
   Memory: 142.2M            
   CGroup: /system.slice/mongodb.service                                                   
           `-10463 /usr/bin/mongod --quiet --config /etc/mongodb.conf                                                                                                                  
                                                                                                                                                                                       
Aug 18 01:22:06 archlinux systemd[1]: Started High-performance, schema-free document-oriented database.                                                                                
Aug 18 01:22:06 archlinux mongod[10463]: 2019-08-18T01:22:06.225+0800 I STORAGE  [main] Max cache overflow file size custom option: 0                                                  
(atomate) [zznu@archlinux atomate]$ mongo                                                                                                                                              
MongoDB shell version v4.0.12                                                              
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb                        
Implicit session: session { "id" : UUID("8331d7de-bf08-49d1-967c-424d90e56992") }                                                                                                      
MongoDB server version: 4.0.12                                                                                                                                                         
> use admin
switched to db admin                                                                                                                  
> db.auth("admin", "admin")                                                                                                                                                            
1                                                                                                                                                                                      
```
**创建atomate数据库**
```
> use atomate                                                                              
switched to db atomate                                                                     
```
**添加可读写权限用户**
```
> db.createUser({user:"atomate",pwd:"atomate",roles:[{role: "readWrite", db: "atomate"}]})                                                                                             
Successfully added user: {                                                                 
        "user" : "atomate",                                                                                                                                                            
        "roles" : [       
                {                                                                          
                        "role" : "readWrite",                                                                                                                                          
                        "db" : "atomate"                                                   
                }                                                                                                                                                                      
        ]                                                                                  
}                                                                                          
```
**添加只读权限用户**
```
> db.createUser({user:"guest",pwd:"guest",roles:[{role: "read", db: "atomate"}]})          
Successfully added user: {   
        "user" : "guest",                                                                  
        "roles" : [                                                                                                                                                                    
                {                                                                                                                                                                      
                        "role" : "read",                                                                                                                                               
                        "db" : "atomate"                                                                                                                                               
                }                                                                                                                                                                      
        ]                                                                                  
}                                                                                          
> exit                                                                                                                                                                                 
bye
```
