#### Set up hadoop enviroment in VMWare:t up hadoop enviroment in VMWare:  
1. Choose Bridge in VM network setting, select Wireless option 
2. Open the tencent.cloud.centos177 image and change the following configuration:
    1. remove ifcfg-lo in /etc/sysconfig/network-scripts 
    2. get the hardware address ->command:  ip addr, something like 00:0c:29:5c:c:f9 
    3. copy the address and paste it after HARDDR in /etc/sysconfig/network-scripts/ifcfg-eth0 
    4. change the IPADDR and GATEWAY in /etc/sysconfig/network-scripts/ifcfg-eth0 
    5. add hostname you want serve datanode in /etc/hosts 
    6. reboot and check wheteher it can connect to network 
3. Install Java jdk

![image](https://github.com/leahReMinD/gs-yarn-basic/pictures/flow.PNG)

#### Background of the flow of YARN application 
          The general concept is that an application submission client submits an application to the YARN ResourceManager (RM). This can be done through setting up a YarnClient object. After YarnClient is started, the client can then set up application context, prepare the very first container of the application that contains the ApplicationMaster (AM), and then submit the application. You need to provide information such as the details about the local files/jars that need to be available for your application to run, the actual command that needs to be executed (with the necessary command line arguments), any OS environment settings (optional), etc. Effectively, you need to describe the Unix process(es) that needs to be launched for your ApplicationMaster.
          The YARN ResourceManager will then launch the ApplicationMaster (as specified) on an allocated container. The ApplicationMaster communicates with YARN cluster, and handles application execution. It performs operations in an asynchronous fashion. During application launch time, the main tasks of the ApplicationMaster are: a) communicating with the ResourceManager to negotiate and allocate resources for future containers, and b) after container allocation, communicating YARN *NodeManager*s (NMs) to launch application containers on them. Task a) can be performed asynchronously through an AMRMClientAsync object, with event handling methods specified in a AMRMClientAsync.CallbackHandler type of event handler. The event handler needs to be set to the client explicitly. Task b) can be performed by launching a runnable object that then launches containers when there are containers allocated. As part of launching this container, the AM has to specify the ContainerLaunchContext that has the launch information such as command line specification, environment, etc.
          During the execution of an application, the ApplicationMaster communicates NodeManagers through NMClientAsync object. All container events are handled by NMClientAsync.CallbackHandler, associated with NMClientAsync. A typical callback handler handles client start, stop, status update and error. ApplicationMaster also reports execution progress to ResourceManager by handling the getProgress() method of AMRMClientAsync.CallbackHandler.
          Other than asynchronous clients, there are synchronous versions for certain workflows (AMRMClient and NMClient). The asynchronous clients are recommended because of (subjectively) simpler usages, and this article will mainly cover the asynchronous clients. Please refer to AMRMClient and NMClient for more information on synchronous clients.
#### Project:
##### This repo contains a barebones implementation of  a YARN app, including :
   -	A YARN client
   -	An Application Master
   -	A container
##### Behaiviour:
   -	This application launches the YARN client 
   -	Which in turn launches the Application master
   -	Which in turn launches a container
   
After mvn clean package in VM, you can run it by java –jar gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-client-0.1.0.jar
And then you can find the output in the logs in hadoop/logs/userlogs/application_xxxx/contain_xxx 
