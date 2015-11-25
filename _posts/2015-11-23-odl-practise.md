---
layout: post
title:  odl-acl-app-explaination
category: project
description:  code explaination about one APP for ODL module
---

## ** refer link **

 * http://sdnhub.org/tutorials/opendaylight/
 * http://www.slideshare.net/sdnhub/opendaylight-app-development-tutoria

## Setup env 

The tutorial application is get from the server https://github.com/sdnhub/SDNHub_Opendaylight_Tutorial.git
it include some sample applications:

  1. a Hub / L2 learning switch,
  2. a network traffic monitoring tap.
  3. acl

<img src="/images/other/odl-sample.png" alt="odl" width="1000"></a>

---

<img src="/images/other/tap.png" alt="odl1" width="1000"></a>

---

here is the command:

    $ git clone https://github.com/sdnhub/SDNHub_Opendaylight_Tutorial.git
    $ cd SDNHub_Opendaylight_Tutorial
    $ git pull --rebase

then do Maven and project building

    $ mvn install -nsu

then do Karaf and feature creation

    $ cd distribution/opendaylight-karaf/target/assembly
    $ ./bin/karaf
    Listening for transport dt_socket at address: 5005                                                                                                                                              
    Hit '' for a list of available commands
    and '[cmd] --help' for help on a specific command.
    Hit '' or type 'system:shutdown' or 'logout' to shutdown OpenDaylight.

    opendaylight-user@root> feature:install odl-mdsal-apidocs odl-dlux-all odl-netconf-connector-all odl-openflowplugin-all
    opendaylight-user@root> feature:install sdnhub-tutorial-learning-switch

    opendaylight-user@root> feature:list | grep sdnhub
    opendaylight-user@root> bundle:list -s | grep sdnhub

    opendaylight-user@root> log:set DEBUG org.sdnhub.odl.tutorial

Note:

To drive karaf, every application built must have a feature description for the karaf shell to load it. The feature.xml file is where we define it.

    $cat features/src/main/resources/feature.xml

### Mininet
we use the sdnhub-tutorial-learning-switch to handle the packet as L2 switch.

    $ sudo mn --topo single,3 --mac --switch ovsk,protocols=OpenFlow13 --controller remote  #in local pc
    $ sudo mn --topo single,3 --mac --switch ovsk,protocols=OpenFlow13 --controller=remote,ip=10.140.179.189
    $ s1 ovs-ofctl add-flow tcp:127.0.0.1:6634 -OOpenFlow13 priority=1,action=output:controller    
    $ sudo ovs-ofctl dump-flows -OOpenFlow13 s1
    $ h1 ping h2 -c 2

show debug log

    opendaylight-user@root> log:display    

before show the log, you can clear the log

    opendaylight-user@root> log:list  
    opendaylight-user@root> log:clear

## tap application
you can config it with the following command:

      curl -H "Content-Type:application/json" -X PUT -d '{"tap-spec":
      {"tap":[
         {"id":"1",
          "node":"openflow:1",
          "vlan-tag":"100",
          "tor-port":"10",
          "access-port":"11"
         },
        {"id":"2",
          "node":"openflow:1",
          "vlan-tag":"100",
          "tor-port":"10",
          "access-port":"11"
         }
        ]
      }
      }' http://10.140.179.189:8181/restconf/config/tap:tap-spec

You can open a browser and inspect the data store at

    http://localhost:8181/restconf/config/tap:tap-spec
    http://localhost:8181/apidoc/explorer/index.html

## acl application

    opendaylight-user@root> feature:install sdnhub-tutorial-acl
    opendaylight-user@root> feature:list | grep sdnhub

---

    curl -H "Content-Type:application/json" -X PUT -d '{"acl-spec":
    {"acl":[
       {"destination":"1.1.1.1",
        "node":"openflow:1",
        "ip-addr":"2.2.2.2/32",
        "port":"10"
       }
      ]
    }
    }' http://10.140.179.189:8181/restconf/config/acl:acl-spec


## APP component

 * Step 1: Define the YANG model for the state
 * Step 2: Activation using config subsystem
	 - Each implementation has a YANG file defining its necessary MD-SAL dependencies and its namespace (See tapapp-impl.yang)
	 - Each module has a config XML file that points to the implementation YANG file along with input to the config subsystem (See 50-tapapp-config.xml)

 * Step 3: Implement the event handlers for all notifications and call backs
	- Provider:
	  + Module that is handles data changes, receives RPC calls, or publishes notifications
	- Consumer:
	  + Module that writes the data, makes RPC calls, or listens to notifications

### Model Module
 * 在model module中定义rpcprovider的Service API（种类，输入输出等）
 * 通过YangToolsPlugin的yang-maven-plugin自动生成Java bindings
 * 生成提供Java bindings的OSGI Bundle

		module acl {
			yang-version 1;
			namespace "urn:sdnhub:odl:tutorial:acl";
			prefix acl;

			import opendaylight-inventory {prefix inv; revision-date 2013-08-19;}
			import yang-ext {prefix ext; revision-date "2013-07-09";}
			import ietf-yang-types { prefix yang; revision-date 2010-09-24; }
			import ietf-inet-types { prefix inet; }
			description "ACL configuration";

			revision "2015-07-22" {
				description "Initial version.";
			}

			typedef port-number {
				type uint32;
			}

			container acl-spec {
				list acl {
					key "destination";
					leaf destination {
						type string;
						}
					leaf node {
						mandatory true;
						type leafref {
							path "/inv:nodes/inv:node/inv:id";
						}
					}
					leaf ip-addr {
						type inet:ipv4-prefix;
					}
					leaf port {
						type port-number;
					}
				}
			}
		}


### config Module

* 通过build-helper-maven-plugin指定向MD-SAL Config Subsystem提供的配置文件信息
* 定义配置文件
* 生成提供配置文件的Jar

		<?xml version="1.0" encoding="UTF-8"?>
		<snapshot>
			<configuration>
			<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
			  <modules xmlns="urn:opendaylight:params:xml:ns:yang:controller:config">
				<module>
				  <type
					  xmlns:prefix="urn:sdnhub:odl:tutorial:acl:acl-impl">prefix:acl-impl</type>
				  <name>acl-impl</name>

			  <notification-service>
				<type xmlns:binding="urn:opendaylight:params:xml:ns:yang:controller:md:sal:binding">binding:binding-notification-service</type>
				<name>binding-notification-broker</name>
			  </notification-service>
			  <data-broker>
				<type xmlns:binding="urn:opendaylight:params:xml:ns:yang:controller:md:sal:binding">binding:binding-async-data-broker</type>
				<name>binding-data-broker</name>
			  </data-broker>
			  <rpc-registry>
				<type xmlns:binding="urn:opendaylight:params:xml:ns:yang:controller:md:sal:binding">binding:binding-rpc-registry</type>
				<name>binding-rpc-broker</name>
			  </rpc-registry>
			</module>
		  </modules>
		</data>
	  </configuration>

	  <required-capabilities>
		<capability>urn:sdnhub:odl:tutorial:acl:acl-impl?module=acl-impl&amp;revision=2015-07-22</capability>
		<capability>urn:opendaylight:params:xml:ns:yang:openflow:common:config:impl?module=openflow-provider-impl&amp;revision=2014-03-26</capability>
	  </required-capabilities>
	</snapshot>

### implementation Module

		module acl-impl {
			yang-version 1;
			namespace "urn:sdnhub:odl:tutorial:acl:acl-impl";
			prefix "acl-impl";

			import config { prefix config; revision-date 2013-04-05; }
			import opendaylight-md-sal-binding { prefix mdsal; revision-date 2013-10-28; }

			description
				"This module contains the base YANG definitions for acl-impl.";

			revision "2015-07-22" {
				description
					"Initial revision.";
			}

			// This is the definition of the service implementation as a module identity
			identity acl-impl {
					base config:module-type;

					// Specifies the prefix for generated java classes.
					config:java-name-prefix TutorialACL;
			}

			// Augments the 'configuration' choice node under modules/module
			augment "/config:modules/config:module/config:configuration" {
				case acl-impl {
					when "/config:modules/config:module/config:type = 'acl-impl'";

					//wires in the data-broker service
					container data-broker {
						uses config:service-ref {
							refine type {
								mandatory true;
								config:required-identity mdsal:binding-async-data-broker;
							}
						}
					}
					container notification-service {
						uses config:service-ref {
						  refine type {
							mandatory false;
							config:required-identity mdsal:binding-notification-service;
						  }
					   }
					}
					container rpc-registry {
						uses config:service-ref {
							refine type {
								mandatory true;
								config:required-identity mdsal:binding-rpc-registry;
							}
						}
					}
				}
			}
		}


Let's do code to handle the event:

        @Override
        public java.lang.AutoCloseable createInstance() {
            //Get all MD-SAL provider objects
            DataBroker dataBroker = getDataBrokerDependency();
            RpcProviderRegistry rpcRegistry = getRpcRegistryDependency();
            NotificationProviderService notificationService = getNotificationServiceDependency();

            TutorialACL tutorialACL = new TutorialACL(dataBroker, notificationService, rpcRegistry);
            LOG.info("Tutorial Access Control List (instance {}) initialized.", tutorialACL);
            return tutorialACL;
        }
        
---
		public class TutorialACL  implements AutoCloseable, DataChangeListener, PacketProcessingListener {
			private final Logger LOG = LoggerFactory.getLogger(this.getClass());
			...
---
the following code is for `listens to notifications`

		public TutorialACL(DataBroker dataBroker, NotificationProviderService notificationService, RpcProviderRegistry rpcProviderRegistry) {
				//Store the data broker for reading/writing from inventory store
				this.dataBroker = dataBroker;

				//Get access to the packet processing service for making RPC calls later
				this.packetProcessingService = rpcProviderRegistry.getRpcService(PacketProcessingService.class);

				//List used to track notification (both data change and YANG-defined) listener registrations
				List<Registration> registrations = Lists.newArrayList();

				InstanceIdentifier<AclSpec> aclSpecIID = InstanceIdentifier.builder(AclSpec.class).build();
                ListenerRegistration<DataChangeListener> registration = dataBroker.registerDataChangeListener(
                    LogicalDatastoreType.CONFIGURATION,aclSpecIID, this, AsyncDataBroker.DataChangeScope.SUBTREE);

               registrations.add(registration);

				//Register this object for receiving notifications when there are PACKET_INs
				registrations.add(notificationService.registerNotificationListener(this));
			}

---
the following interface is defined in `DataChangeListener`

		@Override
		public void onDataChanged(AsyncDataChangeEvent<InstanceIdentifier<?>, DataObject> change) {
			LOG.debug("Data changed: {} created, {} updated, {} removed",
					change.getCreatedData().size(), change.getUpdatedData().size(), change.getRemovedPaths().size());

			DataObject dataObject;

			// Iterate over any created nodes or interfaces
			for (Map.Entry<InstanceIdentifier<?>, DataObject> entry : change.getCreatedData().entrySet()) {
				dataObject = entry.getValue();
				if (dataObject instanceof Acl) {
					programACL((Acl)dataObject);
				}
			}
		}
---
the following interface is defined in `PacketProcessingListener`

		@Override
		public void onPacketReceived(PacketReceived notification) {
			NodeConnectorRef ingressNodeConnectorRef = notification.getIngress();
			NodeRef ingressNodeRef = InventoryUtils.getNodeRef(ingressNodeConnectorRef);
			// NodeConnectorId ingressNodeConnectorId = InventoryUtils.getNodeConnectorId(ingressNodeConnectorRef);
			NodeId ingressNodeId = InventoryUtils.getNodeId(ingressNodeConnectorRef);

			// Useful to create it beforehand
			NodeConnectorId floodNodeConnectorId = InventoryUtils.getNodeConnectorId(ingressNodeId, FLOOD_PORT_NUMBER);
			NodeConnectorRef floodNodeConnectorRef = InventoryUtils.getNodeConnectorRef(floodNodeConnectorId);

			//Ignore LLDP packets, or you will be in big trouble
			byte[] etherTypeRaw = PacketParsingUtils.extractEtherType(notification.getPayload());
			int etherType = (0x0000ffff & ByteBuffer.wrap(etherTypeRaw).getShort());
			if (etherType == 0x88cc) {
				return;
			}

			// Flood packet
			packetOut(ingressNodeRef, floodNodeConnectorRef, notification.getPayload());
		}

			
		}    
		
 
Tap's code to handle the event:

---
        
	public class TutorialTapProvider implements AutoCloseable, DataChangeListener, OpendaylightInventoryListener {
		private final Logger LOG = LoggerFactory.getLogger(this.getClass());
        ...
     }
     
---

    public TutorialTapProvider(DataBroker dataBroker, NotificationProviderService notificationService, RpcProviderRegistry rpcProviderRegistry) {
        //Store the data broker for reading/writing from inventory store
        this.dataBroker = dataBroker;

        //Object used for flow programming through RPC calls
        this.salFlowService = rpcProviderRegistry.getRpcService(SalFlowService.class);

        //List used to track notification (both data change and YANG-defined) listener registrations
        this.registrations = registerDataChangeListeners();

        //Register this object for receiving notifications when there are New switches
        registrations.add(notificationService.registerNotificationListener(this));
    }   
    
---
the following interface is defined in `OpendaylightInventoryListener`

    @Override
    public void onNodeRemoved(NodeRemoved nodeRemoved) {
        LOG.debug("Node removed {}", nodeRemoved);
    }

    @Override
    public void onNodeConnectorRemoved(NodeConnectorRemoved nodeConnectorRemoved) {
        LOG.debug("Node connector removed {}", nodeConnectorRemoved);
    }

    @Override
    public void onNodeUpdated(NodeUpdated nodeUpdated) {
        LOG.debug("Node updated {}", nodeUpdated);
    }
    
---            

### Load the feature

There are also commands to install a feature (feature:install) or a specific bundle (bundle:install).  

By default karaf loads all the features listed in the  
   distribution/opendaylight-karaf/target/assembly/etc/org.apache.karaf.features.cfg file.  
In our example, we autoload sdnhub-tutorial-tapapp feature.

    $ cat features/src/main/resources/feature.xml
    ...
    <feature name="sdnhub-tutorial-tapapp" description="SDN Hub Tutorial :: OpenDaylight :: Tap application" version='${project.version}'>
        <feature version="${openflowplugin.version}">odl-openflowplugin-southbound</feature>
        <feature version="${openflowplugin.version}">odl-openflowplugin-flow-services</feature>
        <feature version="${mdsal.version}">odl-mdsal-broker</feature>
        <bundle>mvn:org.sdnhub.odl.tutorial.tapapp/tapapp-impl/${tapapp.version}</bundle>
        <bundle>mvn:org.sdnhub.odl.tutorial.tapapp/tapapp-model/${tapapp.version}</bundle>
        <configfile finalname="etc/opendaylight/karaf/${tapapp.configfile}">mvn:org.sdnhub.odl.tutorial.tapapp/tapapp-config/${tapapp.version}/xml/config</configfile>
    </feature>
    ...

---
I comment the content about sdnhub-tutorial-acl in the file features/src/main/resources/feature.xml  
use the following command to install it:

	opendaylight-user@root> install file:/C:/sdn/SDNHub_Opendaylight_Tutorial/acl/model/target/acl-model-1.0.0-SNAPSHOT.jar
	opendaylight-user@root> install file:/C:/sdn/SDNHub_Opendaylight_Tutorial/acl/implementation/target/acl-impl-1.0.0-SNAPSHOT.jar
	opendaylight-user@root> install file:/C:/sdn/SDNHub_Opendaylight_Tutorial/acl/config/target/acl-config-1.0.0-SNAPSHOT.jar    
	opendaylight-user@root> start x
	opendaylight-user@root> start y
	opendaylight-user@root> start z
    `x y z` is the bundle id.
    
  
