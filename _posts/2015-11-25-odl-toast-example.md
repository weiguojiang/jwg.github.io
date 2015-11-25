---
layout: post
title:  odl-toast-app-stey-by-step
category: project
description:   ODL controller example for toast step by step
---

## Introduction
Here list the code to make you understand the module of the APP.

## refer link

 * https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Toaster_Step-By-Step
 * https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Toaster_Tutorial

## Part 1:  north-bound interface to get operational data

 * Define the toaster data model (north-bound interface)
 * Provide a read-only implementation to retrieve operational data on the toaster.

---
### yang model

		//This file contains a YANG data definition. This data model defines
		  //a toaster, which is based on the SNMP MIB Toaster example 
		  module toaster {

			//The yang version - today only 1 version exists. If omitted defaults to 1.
			yang-version 1; 

			//a unique namespace for this toaster module, to uniquely identify it from other modules that may have the same name.
			namespace
			  "http://netconfcentral.org/ns/toaster"; 

			//a shorter prefix that represents the namespace for references used below
			prefix toast;

			//Defines the organization which defined / owns this .yang file.
			organization "Netconf Central";

			//defines the primary contact of this yang file.
			contact
			  "Andy Bierman <andy@netconfcentral.org>";

			//provides a description of this .yang file.
			description
			  "YANG version of the TOASTER-MIB.";

			//defines the dates of revisions for this yang file
			revision "2009-11-20" {
			  description
				"Toaster module in progress.";
			}

			//declares a base identity, in this case a base type for different types of toast.
			identity toast-type {
			  description
				"Base for all bread types supported by the toaster. New bread types not listed here nay be added in the future.";
			}

			//the below identity section is used to define globally unique identities
			//Note - removed a number of different types of bread to shorten the text length.
			identity white-bread {
			  base toast:toast-type;       //logically extending the declared toast-type above.
			  description "White bread.";  //free text description of this type.
			}

			identity wheat-bread {
			  base toast-type;
			  description "Wheat bread.";
			}

			//defines a new "Type" string type which limits the length
			typedef DisplayString {
			  type string {
				length "0 .. 255";
			  }
			  description
				"YANG version of the SMIv2 DisplayString TEXTUAL-CONVENTION.";
			  reference
				"RFC 2579, section 2.";

			}

			// This definition is the top-level configuration "item" that defines a toaster. The "presence" flag connotes there
			// can only be one instance of a toaster which, if present, indicates the service is available.
			container toaster {
			  presence
				"Indicates the toaster service is available";
			  description
				"Top-level container for all toaster database objects.";

			  //Note in these three attributes that config = false. This indicates that they are operational attributes.
			  leaf toasterManufacturer {
				type DisplayString;
				config false;
				mandatory true;
				description
				  "The name of the toaster's manufacturer. For instance, Microsoft Toaster.";
			  }

			  leaf toasterModelNumber {
				type DisplayString;
				config false;
				mandatory true;
				description
				  "The name of the toaster's model. For instance, Radiant Automatic.";
			  }

			  leaf toasterStatus {
				type enumeration {
				  enum "up" {
					value 1;
					description
					  "The toaster knob position is up. No toast is being made now.";
				  }
				  enum "down" {
					value 2;
					description
					  "The toaster knob position is down. Toast is being made now.";
				  }
				}
				config false;
				mandatory true;
				description
				  "This variable indicates the current state of  the toaster.";
			  }
			}  // container toaster
		  }  // module toaster

---
### impl model

		module toaster-provider-impl {
			yang-version 1;
			namespace "urn:opendaylight:params:xml:ns:yang:controller:config:toaster-provider:impl";
			prefix "toaster-provider-impl";

			import config { prefix config; revision-date 2013-04-05; }
			import opendaylight-md-sal-binding { prefix mdsal; revision-date 2013-10-28; }

			description
				"This module contains the base YANG definitions for toaster-provider impl implementation.";

			revision "2014-01-31" {
				description
					"Initial revision.";
			}

			// This is the definition of the service implementation as a module identity
			identity toaster-provider-impl {
					base config:module-type;

					// Specifies the prefix for generated java classes.
					config:java-name-prefix ToasterProvider;
			}

			// Augments the 'configuration' choice node under modules/module.  
			augment "/config:modules/config:module/config:configuration" {
				case toaster-provider-impl {
					when "/config:modules/config:module/config:type = 'toaster-provider-impl'";

					//wires in the data-broker service 
					container data-broker {
						uses config:service-ref {
							refine type {
								mandatory false;
								config:required-identity mdsal:binding-async-data-broker;
							}
						}
					}      
				}
			}
		}
---
### config model

		<snapshot>
			
			<configuration>
				<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
					<modules xmlns="urn:opendaylight:params:xml:ns:yang:controller:config">

						<!-- defines an implementation module -->
						<module>
							<type xmlns:toaster="urn:opendaylight:params:xml:ns:yang:controller:config:toaster-provider:impl">
								toaster:toaster-provider-impl
							</type>
							
							<name>toaster-provider-impl</name>

							<data-broker>
							   <type xmlns:binding="urn:opendaylight:params:xml:ns:yang:controller:md:sal:binding">binding:binding-async-data-broker</type>
							   <name>binding-data-broker</name>
						   </data-broker>
						</module>

					</modules>
				</data>
			</configuration>
		  
			<required-capabilities>
				<capability>urn:opendaylight:params:xml:ns:yang:controller:config:toaster-provider:impl?module=toaster-provider-impl&amp;revision=2014-01-31</capability>
			</required-capabilities>
		  
		</snapshot>

---
### feature model

	  <feature name='odl-toaster' version='${project.version}' description="OpenDaylight :: Toaster">
		   <feature version='${yangtools.version}'>odl-yangtools-common</feature>
		   <feature version='${yangtools.version}'>odl-yangtools-binding</feature>
		   <feature version='${project.version}'>odl-mdsal-broker</feature>
		   <bundle>mvn:org.opendaylight.controller.samples/toaster/${project.version}</bundle>
		   <bundle>mvn:org.opendaylight.controller.samples/toaster-consumer/${project.version}</bundle>
		   <bundle>mvn:org.opendaylight.controller.samples/toaster-provider/${project.version}</bundle>
		   <configfile finalname="${config.configfile.directory}/${config.toaster.configfile}">mvn:org.opendaylight.controller.samples/toaster-config/${project.version}/xml/config</configfile>
	   </feature>

---
### code

		public class OpendaylightToaster implements AutoCloseable{
		  
		   //making this public because this unique ID is required later on in other classes.
		   public static final InstanceIdentifier<Toaster>  TOASTER_IID = InstanceIdentifier.builder(Toaster.class).build();
			  
		   private static final DisplayString TOASTER_MANUFACTURER = new DisplayString("Opendaylight");
		   private static final DisplayString TOASTER_MODEL_NUMBER = new DisplayString("Model 1 - Binding Aware");
			
		   private DataBroker dataProvider;
		  
		   public OpendaylightToaster() {
		   }
			
		   private Toaster buildToaster( ToasterStatus status ) {
			   
			   // note - we are simulating a device whose manufacture and model are
			   // fixed (embedded) into the hardware.
			   // This is why the manufacture and model number are hardcoded.
			   return new ToasterBuilder().setToasterManufacturer( TOASTER_MANUFACTURER )
										  .setToasterModelNumber( TOASTER_MODEL_NUMBER )
										  .setToasterStatus( status )
										  .build();
		   }
		   
		   public void setDataProvider( final DataBroker salDataProvider ) {
				this.dataProvider = salDataProvider;
				setToasterStatusUp( null );
		   }
		 
		   /**
			* Implemented from the AutoCloseable interface.
			*/
		   @Override
		   public void close() throws ExecutionException, InterruptedException {
			   if (dataProvider != null) {
				   WriteTransaction t = dataProvider.newWriteOnlyTransaction();
				   t.delete(LogicalDatastoreType.OPERATIONAL,TOASTER_IID);
				   ListenableFuture<RpcResult<TransactionStatus>> future = t.commit();
				   Futures.addCallback( future, new FutureCallback<RpcResult<TransactionStatus>>() {
					   @Override
					   public void onSuccess( RpcResult<TransactionStatus> result ) {
						   LOG.debug( "Delete Toaster commit result: " + result );
					   }
					   
					   @Override
					   public void onFailure( Throwable t ) {
						   LOG.error( "Delete of Toaster failed", t );
					   }
				   } );
			   }
		   }
		   
		   private void setToasterStatusUp( final Function<Boolean,Void> resultCallback ) {
			   
			   WriteTransaction tx = dataProvider.newWriteOnlyTransaction();
			   tx.put( LogicalDatastoreType.OPERATIONAL,TOASTER_IID, buildToaster( ToasterStatus.Up ) );
			   
			   ListenableFuture<RpcResult<TransactionStatus>> commitFuture = tx.commit();
			   
			   Futures.addCallback( commitFuture, new FutureCallback<RpcResult<TransactionStatus>>() {
				   @Override
				   public void onSuccess( RpcResult<TransactionStatus> result ) {
					   if( result.getResult() != TransactionStatus.COMMITED ) {
						   LOG.error( "Failed to update toaster status: " + result.getErrors() );
					   }
					   
					   notifyCallback( result.getResult() == TransactionStatus.COMMITED );
				   }
				   
				   @Override
				   public void onFailure( Throwable t ) {
					   // We shouldn't get an OptimisticLockFailedException (or any ex) as no
					   // other component should be updating the operational state.
					   LOG.error( "Failed to update toaster status", t );
					   
					   notifyCallback( false );
				   }
				   
				   void notifyCallback( boolean result ) {
					   if( resultCallback != null ) {
						   resultCallback.apply( result );
					   }
				   }
			   } );
		   }
		}
---
### test

		URL: http://localhost:8080/restconf/operational/toaster:toaster

		{
			toaster: {
				toasterManufacturer: "Opendaylight"
				toasterModelNumber: "Model 1 - Binding Aware"
				toasterStatus: "Up"
		   }
		}

---


## Part 2: RPC call 
 
Add and implement a remote procedure call which will allow the user to interact  
with the operations restconf interface, as well as see status changes to operational data.

---
### yang model

		module toaster {
			   ... 
		   //This defines a Remote Procedure Call (rpc). RPC provide the ability to initiate an action
		   //on the data model. In this case the initating action takes two optional inputs (because default value is defined)
		   //QUESTION: Am I correct that the inputs are optional because they have defaults defined? The REST call doesn't seem to account for this.
		   rpc make-toast {
			 description
			   "Make some toast. The toastDone notification will be sent when the toast is finished.
				An 'in-use' error will be returned if toast is already being made. A 'resource-denied' error will 
				be returned if the toaster service is disabled.";

			 input {
			   leaf toasterDoneness {
				 type uint32 {
				   range "1 .. 10";
				 }
				 default '5';
				 description
				   "This variable controls how well-done is the ensuing toast. It should be on a scale of 1 to 10.
					Toast made at 10 generally is considered unfit for human consumption; toast made at 1 is warmed lightly.";
			   }

			   leaf toasterToastType {
				 type identityref {
				   base toast:toast-type;
				 }
				 default 'wheat-bread';
				 description
				   "This variable informs the toaster of the type of material that is being toasted. The toaster uses this information, 
					 combined with toasterDoneness, to compute for how long the material must be toasted to achieve the required doneness.";
			   }
			 }
		   }  // rpc make-toast

		   // action to cancel making toast - takes no input parameters
		   rpc cancel-toast {
			 description
			   "Stop making toast, if any is being made.
				  A 'resource-denied' error will be returned 
				  if the toaster service is disabled.";
		   }  // rpc cancel-toast
		   ...
		 }
	
---
### impl model

   //augments the configuration,  
   augment "/config:modules/config:module/config:configuration" {
       case toaster-provider-impl {
           when "/config:modules/config:module/config:type = 'toaster-provider-impl'";
           ...     
           
           //Wires dependent services into this class - in this case the RPC registry service
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
   	
---
### config model

	 <module>
		 <type xmlns:prefix="urn:opendaylight:params:xml:ns:yang:controller:config:toaster-provider:impl">
			   prefix:toaster-provider-impl
		  </type>
		 
		  <name>toaster-provider-impl</name>
		 
		   <rpc-registry>
				  <type xmlns:binding="urn:opendaylight:params:xml:ns:yang:controller:md:sal:binding">binding:binding-rpc-registry</type>
				  <name>binding-rpc-broker</name>
		   </rpc-registry>
	   
		   ...
	  
	  </module>
	
---
### feature model

	
---
### code model

	  @Override
	   public java.lang.AutoCloseable createInstance() {
		   final OpendaylightToaster opendaylightToaster = new OpendaylightToaster();
	   
		   ...
		   
		   final BindingAwareBroker.RpcRegistration<ToasterService> rpcRegistration = getRpcRegistryDependency()
				   .addRpcImplementation(ToasterService.class, opendaylightToaster);
			  
		   final class AutoCloseableToaster implements AutoCloseable {
		
			   @Override
			   public void close() throws Exception {
				   ...
				   rpcRegistration.close();
				   ...
			   }
	  
		   }
	   
		   return new AutoCloseableToaster();
	   }
---
	   
		public class OpendaylightToaster implements ToasterService, AutoCloseable {
		 
		  ...  
		  private final ExecutorService executor;
		  
		  // The following holds the Future for the current make toast task.
		  // This is used to cancel the current toast.
		  private final AtomicReference<Future<?>> currentMakeToastTask = new AtomicReference<>();
		  
		  public OpendaylightToaster() {
			  executor = Executors.newFixedThreadPool(1);
		  }
		   
		  /**
		  * Implemented from the AutoCloseable interface.
		  */
		  @Override
		  public void close() throws ExecutionException, InterruptedException {
			  // When we close this service we need to shutdown our executor!
			  executor.shutdown();
			  
			  ...
		  }
		  
		  @Override
		  public Future<RpcResult<Void>> cancelToast() {
		  
			  Future<?> current = currentMakeToastTask.getAndSet( null );
			  if( current != null ) {
				  current.cancel( true );
			  }
		 
			  // Always return success from the cancel toast call.
			  return Futures.immediateFuture( Rpcs.<Void> getRpcResult( true,
											  Collections.<RpcError>emptyList() ) );
		  }
			
		  @Override
		  public Future<RpcResult<Void>> makeToast(final MakeToastInput input) {
			  final SettableFuture<RpcResult<Void>> futureResult = SettableFuture.create();
		 
			  checkStatusAndMakeToast( input, futureResult );
		 
			  return futureResult;
		  }
		 
		  private void checkStatusAndMakeToast( final MakeToastInput input,
												final SettableFuture<RpcResult<Void>> futureResult ) {
		 
			  // Read the ToasterStatus and, if currently Up, try to write the status to Down.
			  // If that succeeds, then we essentially have an exclusive lock and can proceed
			  // to make toast.
		 
			  final ReadWriteTransaction tx = dataProvider.newReadWriteTransaction();
			  ListenableFuture<Optional<DataObject>> readFuture =
												 tx.read( LogicalDatastoreType.OPERATIONAL, TOASTER_IID );
		 
			  final ListenableFuture<RpcResult<TransactionStatus>> commitFuture =
				  Futures.transform( readFuture, new AsyncFunction<Optional<DataObject>,
																		  RpcResult<TransactionStatus>>() {
		 
					  @Override
					  public ListenableFuture<RpcResult<TransactionStatus>> apply(
							  Optional<DataObject> toasterData ) throws Exception {
		 
						  ToasterStatus toasterStatus = ToasterStatus.Up;
						  if( toasterData.isPresent() ) {
							  toasterStatus = ((Toaster)toasterData.get()).getToasterStatus();
						  }
		 
						  LOG.debug( "Read toaster status: {}", toasterStatus );
		 
						  if( toasterStatus == ToasterStatus.Up ) {
		 
							  LOG.debug( "Setting Toaster status to Down" );
		 
							  // We're not currently making toast - try to update the status to Down
							  // to indicate we're going to make toast. This acts as a lock to prevent
							  // concurrent toasting.
							  tx.put( LogicalDatastoreType.OPERATIONAL, TOASTER_IID,
									  buildToaster( ToasterStatus.Down ) );
							  return tx.commit();
						  }
		 
						  LOG.debug( "Oops - already making toast!" );
		 
						  // Return an error since we are already making toast. This will get
						  // propagated to the commitFuture below which will interpret the null
						  // TransactionStatus in the RpcResult as an error condition.
						  return Futures.immediateFuture( Rpcs.<TransactionStatus>getRpcResult(
								  false, null, makeToasterInUseError() ) );
					  }
			  } );
		 
			  Futures.addCallback( commitFuture, new FutureCallback<RpcResult<TransactionStatus>>() {
				  @Override
				  public void onSuccess( RpcResult<TransactionStatus> result ) {
					  if( result.getResult() == TransactionStatus.COMMITED  ) {
		 
						  // OK to make toast
						  currentMakeToastTask.set( executor.submit(
														   new MakeToastTask( input, futureResult ) ) );
					  } else {
		 
						  LOG.debug( "Setting error result" );
		 
						  // Either the transaction failed to commit for some reason or, more likely,
						  // the read above returned ToasterStatus.Down. Either way, fail the
						  // futureResult and copy the errors.
		 
						  futureResult.set( Rpcs.<Void>getRpcResult( false, null, result.getErrors() ) );
					  }
				  }
		 
				  @Override
				  public void onFailure( Throwable ex ) {
					  if( ex instanceof OptimisticLockFailedException ) {
		 
						  // Another thread is likely trying to make toast simultaneously and updated the
						  // status before us. Try reading the status again - if another make toast is
						  // now in progress, we should get ToasterStatus.Down and fail.
		 
						  LOG.debug( "Got OptimisticLockFailedException - trying again" );
		 
						  checkStatusAndMakeToast( input, futureResult );
		 
					  } else {
		 
						  LOG.error( "Failed to commit Toaster status", ex );
		 
						  // Got some unexpected error so fail.
						  futureResult.set( Rpcs.<Void> getRpcResult( false, null, Arrays.asList(
							   RpcErrors.getRpcError( null, null, null, ErrorSeverity.ERROR,
													  ex.getMessage(),
													  ErrorType.APPLICATION, ex ) ) ) );
					  }
				  }
			  } );
		  }
		 
		  private class MakeToastTask implements Callable<Void> {
		 
			  final MakeToastInput toastRequest;
			  final SettableFuture<RpcResult<Void>> futureResult;
		 
			  public MakeToastTask( final MakeToastInput toastRequest,
									final SettableFuture<RpcResult<Void>> futureResult ) {
				  this.toastRequest = toastRequest;
				  this.futureResult = futureResult;
			  }
		 
			  @Override
			  public Void call() {
				  try
				  {
					  // make toast just sleeps for n seconds.
					  long darknessFactor = OpendaylightToaster.this.darknessFactor.get();
					  Thread.sleep(toastRequest.getToasterDoneness());
				  }
				  catch( InterruptedException e ) {
					  LOG.info( "Interrupted while making the toast" );
				  }
		 
				  toastsMade.incrementAndGet();
		 
				  amountOfBreadInStock.getAndDecrement();
				  if( outOfBread() ) {
					  LOG.info( "Toaster is out of bread!" );
		 
					  notificationProvider.publish( new ToasterOutOfBreadBuilder().build() );
				  }
		 
				  // Set the Toaster status back to up - this essentially releases the toasting lock.
				  // We can't clear the current toast task nor set the Future result until the
				  // update has been committed so we pass a callback to be notified on completion.
		 
				  setToasterStatusUp( new Function<Boolean,Void>() {
					  @Override
					  public Void apply( Boolean result ) {
		 
						  currentMakeToastTask.set( null );
		 
						  LOG.debug("Toast done");
		 
						  futureResult.set( Rpcs.<Void>getRpcResult( true, null,
																 Collections.<RpcError>emptyList() ) );
		 
						  return null;
					  }
				  } );
				  return null;
			 }
		}
	 
---
### test model

		HTTP Method => POST
		URL => http://localhost:8080/restconf/operations/toaster:make-toast 
		Header =>   Content-Type: application/yang.data+json  
		Body =>  
		{
		  "input" :
		  {
			 "toaster:toasterDoneness" : "10",
			 "toaster:toasterToastType":"wheat-bread" 
		  }
		}
---

		   toaster: {
			   toasterManufacturer: "Opendaylight"
			   toasterModelNumber: "Model 1 - Binding Aware"
			   toasterStatus: "Down"
		  }
		

---

## Part 3: modify data and lister

 * Illustrates how a user can modify configuration data via restconf
 * how our toaster can listener for those changes.

---
### yang model

		container toaster {
			 ...
			
			 leaf darknessFactor {
			   type uint32;
			   config true;
			   default 1000;
			   description
				 "The darkness factor. Basically, the number of ms to multiple the doneness value by.";
			 }
			
			 ...
		}
	
---
### impl model

	
---
### config model

	
---
### feature model

	
---
### code model

	  @Override
	   public java.lang.AutoCloseable createInstance() {
		   final OpendaylightToaster opendaylightToaster = new OpendaylightToaster();
			
		   ...
		   
		   final ListenerRegistration<DataChangeListener> dataChangeListenerRegistration = 
				   dataBrokerService.registerDataChangeListener( OpendaylightToaster.TOASTER_IID, opendaylightToaster );
		   

		   ...
		   final class AutoCloseableToaster implements AutoCloseable {     
			   @Override
			   public void close() throws Exception {
				   dataChangeListenerRegistration.close(); //closes the listener registrations (removes it)
				   ...
			   }
		   }
		   ...
	   }
	   
---
		  ...
		  import org.opendaylight.controller.sal.binding.api.data.DataChangeListener;
		  ...
		  public class OpendaylightToaster implements ToasterData, ToasterService, AutoCloseable, DataChangeListener {
		  ...
			 @Override
			 public void onDataChanged(DataChangeEvent<InstanceIdentifier<?>, DataObject> change) {
					 //TODO - implement
			 }
		  ...
		  }
		  
---
		 ...
		  //Thread safe holder for our darkness multiplier.
		  private AtomicLong darknessFactor = new AtomicLong( 1000 );
		  ...
		  @Override
		  public void onDataChanged(DataChangeEvent<InstanceIdentifier<?>, DataObject> change) {
			  DataObject dataObject = change.getUpdatedSubtree();
			  if( dataObject instanceof Toaster )
			  {
				  Toaster toaster = (Toaster) dataObject;
				  Long darkness = toaster.getDarknessFactor();
				  if( darkness != null )
				  {
					  darknessFactor.set( darkness );
				  }
			  }
		  }
		   ...
	 
---
### test model

	  HTTP Method: PUT
	  URL:  http://localhost:8080/restconf/config/toaster:toaster
	  HEADER: content-type: application/yang.data+json
	  BODY: 
	  {
		toaster:
		{
		   darknessFactor: "2000"
		 }
	   }
		
---

## Part 4: internal data by JMX: NOT from north-bound interface
 
 * Provide additional statistical attributes not present in the north bound interface 
 * but available by the implementation via JMX.

---
### yang model

	
---
### impl model

		import rpc-context { prefix rpcx; revision-date 2013-06-17; }
		...
		augment "/config:modules/config:module/config:state" {
			case toaster-provider-impl {
				when "/config:modules/config:module/config:type = 'toaster-provider-impl'";

				leaf toasts-made {
					type uint32;
				}

				rpcx:rpc-context-instance "clear-toasts-made-rpc";
			}
		}

		identity clear-toasts-made-rpc;

		rpc clear-toasts-made  {
			description
			  "JMX call to clear the toasts-made counter.";

			input {
				uses rpcx:rpc-context-ref {
					refine context-instance {
						rpcx:rpc-context-instance clear-toasts-made-rpc;
					}
				}
			}
		}
	
---
### config model

	
---
### feature model

	
---
### code model

	   public java.lang.AutoCloseable createInstance() {
		   final OpendaylightToaster opendaylightToaster = new OpendaylightToaster();
		   ...
		   // Register runtimeBean for toaster statistics via JMX
		   final ToasterProviderRuntimeRegistration runtimeReg = getRootRuntimeBeanRegistratorWrapper().register( opendaylightToaster);
		   ...
		   final class AutoCloseableToaster implements AutoCloseable {
			   @Override
			   public void close() throws Exception {
				   ...
				   runtimeReg.close();
				   ...
			   }
			   ...
		   }
	   }	
---
		public class OpendaylightToaster implements ToasterService, AutoCloseable, DataChangeListener, ToasterProviderRuntimeMXBean {
		   ...
		   private final AtomicLong toastsMade = new AtomicLong(0);
		   ...
		   

		   /**
			* Accessor method implemented from the ToasterProviderRuntimeMXBean interface.
			*/
		   @Override
		   public Long getToastsMade() {
			   return toastsMade.get();
		   }
		   

		   /**
			* JMX RPC call implemented from the ToasterProviderRuntimeMXBean interface.
			*/
		   @Override
		   public void clearToastsMade() {
			   LOG.info( "clearToastsMade" );
			   toastsMade.set( 0 );
		   }
		   ...
		   

		   private class MakeToastTask implements Callable<Void> {
			   ...
			   @Override
			   public Void call() throws InterruptedException {
				   ...
				   toastsMade.incrementAndGet();
				   ...
			   }
		   }
		}
	 
---
### test model

    ./run.sh -jmx 

---
    $JAVA_HOME/bin/jconsole
	
    Once connected, navigate to the "MBeans" tab.

    Expand the "org.opendaylight.controller->RuntimeBean->toaster-provider-impl->toster-provider-impl" nodes.

    Select "Attributes". You will now see the "ToastsMade" attribute displayed and this attribute will change when the make-toast RPC call is executed.
    After you have called ToastsMade call a few times, refresh the attributes and see that the value increased.
    Now select "Operations", and click the "clearToastsMade" button.
    Return to the Attributes and note that the counter is now set to 0.
	

---

## Part 5: service chain

 * Introduce a KitchenService that is a consumer of the toaster model. 
 * This provides a demonstration of how other business intelligence in the controller can access the data models 
 * and invoke RPC calls for the purpose of providing additional business logic in the controller.

---
### yang model

	
---
### impl model

		module kitchen-service-impl {

			yang-version 1;
			namespace "urn:opendaylight:params:xml:ns:yang:controller:config:kitchen-service:impl";
			prefix "kitchen-service-impl";

			import config { prefix config; revision-date 2013-04-05; }
			import rpc-context { prefix rpcx; revision-date 2013-06-17; }

			import opendaylight-md-sal-binding { prefix mdsal; revision-date 2013-10-28; }

			description
				"This module contains the base YANG definitions for
				kitchen-service impl implementation.";

			revision "2014-01-31" {
				description
					"Initial revision.";
			}

			// This is the definition of kitchen service interface identity.
			identity kitchen-service {
				base "config:service-type";
				config:java-class "org.opendaylight.controller.sample.kitchen.api.KitchenService";
			}

			// This is the definition of kitchen service implementation module identity. 
			identity kitchen-service-impl {
					base config:module-type;
					config:provided-service kitchen-service;
					config:java-name-prefix KitchenService;
			}

			augment "/config:modules/config:module/config:configuration" {
				case kitchen-service-impl {
					when "/config:modules/config:module/config:type = 'kitchen-service-impl'";

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
			
---
### config model

		<snapshot>
		   <configuration>
			   
				   <modules xmlns="urn:opendaylight:params:xml:ns:yang:controller:config">
					  ...
					  <module>
						 <type xmlns:kitchen="urn:opendaylight:params:xml:ns:yang:controller:config:kitchen-service:impl">
							kitchen:kitchen-service-impl
						 </type>
						 <name>kitchen-service-impl</name>
						 

						 <rpc-registry>
							<type xmlns:binding="urn:opendaylight:params:xml:ns:yang:controller:md:sal:binding">binding:binding-rpc-registry</type>
							<name>binding-rpc-broker</name>
						 </rpc-registry>
					   </module>
				   </modules>
				   <services xmlns="urn:opendaylight:params:xml:ns:yang:controller:config">
					   <service>
						   <type xmlns:kitchen="urn:opendaylight:params:xml:ns:yang:controller:config:kitchen-service:impl">
							   kitchen:kitchen-service
						   </type>
						   <instance>
							   <name>kitchen-service</name>
							   <provider>/modules/module[type='kitchen-service-impl'][name='kitchen-service-impl']</provider>
						   </instance>
					   </service>
				   </services>
			   
		   </configuration>
		   
		   <required-capabilities>
			   <capability>urn:opendaylight:params:xml:ns:yang:controller:config:kitchen-service:impl?module=kitchen-service-impl&amp;revision=2014-01-31</capability>
			   <capability>urn:opendaylight:params:xml:ns:yang:controller:config:toaster-provider:impl?module=toaster-provider-impl&amp;revision=2014-01-31</capability>
		   </required-capabilities>
		</snapshot>
	
---
### feature model

	
---
### code model

---
    @Override
    public java.lang.AutoCloseable createInstance() {
        ToasterService toasterService = getRpcRegistryDependency().getRpcService(ToasterService.class);

        final KitchenServiceImpl kitchenService = new KitchenServiceImpl(toasterService);

        final class AutoCloseableKitchenService implements KitchenService, AutoCloseable {

            @Override
            public void close() throws Exception {
            }

            @Override
            public Future<RpcResult<Void>> makeBreakfast( EggsType eggs, Class<? extends ToastType> toast, int toastDoneness ) {
                return kitchenService.makeBreakfast( eggs, toast, toastDoneness );
            }
        }

        AutoCloseable ret = new AutoCloseableKitchenService();
        return ret;
    }

---
	//KitchenService.java 
	public interface KitchenService {
	  
		Future<RpcResult<Void>> makeBreakfast( EggsType eggs, Class<? extends ToastType> toast, int toastDoneness );
	   
	}
---
		public class KitchenServiceImpl implements KitchenService {

			private static final Logger log = LoggerFactory.getLogger( KitchenServiceImpl.class );

			private final ToasterService toaster;

			public KitchenServiceImpl(ToasterService toaster) {
				this.toaster = toaster;
			}

			@Override
			public Future<RpcResult<Void>> makeBreakfast( EggsType eggs, Class<? extends ToastType> toast, int toastDoneness ) {
		  
				// Call makeToast and use JdkFutureAdapters to convert the Future to a ListenableFuture,
				// The OpendaylightToaster impl already returns a ListenableFuture so the conversion is
				// actually a no-op.
		  
				ListenableFuture<RpcResult<Void>> makeToastFuture = JdkFutureAdapters.listenInPoolThread(
						makeToast( toastType, toastDoneness ), executor );
		  
				ListenableFuture<RpcResult<Void>> makeEggsFuture = makeEggs( eggsType );
		  
				// Combine the 2 ListenableFutures into 1 containing a list of RpcResults.
		  
				ListenableFuture<List<RpcResult<Void>>> combinedFutures =
						Futures.allAsList( ImmutableList.of( makeToastFuture, makeEggsFuture ) );
		  
				// Then transform the RpcResults into 1.
		  
				return Futures.transform( combinedFutures,
					new AsyncFunction<List<RpcResult<Void>>,RpcResult<Void>>() {
						@Override
						public ListenableFuture<RpcResult<Void>> apply( List<RpcResult<Void>> results )
																						 throws Exception {
							boolean atLeastOneSucceeded = false;
							Builder<RpcError> errorList = ImmutableList.builder();
							for( RpcResult<Void> result: results ) {
								if( result.isSuccessful() ) {
									atLeastOneSucceeded = true;
								}
		  
								if( result.getErrors() != null ) {
									errorList.addAll( result.getErrors() );
								}
							}
		  
							return Futures.immediateFuture(
									  Rpcs.<Void> getRpcResult( atLeastOneSucceeded, errorList.build() ) );
						}
				} );
			}
		  
			private ListenableFuture<RpcResult<Void>> makeEggs( EggsType eggsType ) {
		  
				return executor.submit( new Callable<RpcResult<Void>>() {
		  
					@Override
					public RpcResult<Void> call() throws Exception {
		  
						// We don't actually do anything here - just return a successful result.
						return Rpcs.<Void> getRpcResult( true, Collections.<RpcError>emptyList() );
					}
				} );
			}
		  
			private Future<RpcResult<Void>> makeToast( Class<? extends ToastType> toastType,
													   int toastDoneness ) {
				// Access the ToasterService to make the toast.
		  
				MakeToastInput toastInput = new MakeToastInputBuilder()
					.setToasterDoneness( (long) toastDoneness )
					.setToasterToastType( toastType )
					.build();
		  
				return toaster.makeToast( toastInput );
			}
		}

---
	
	 
---
### test model


### Add JMX RPC to make breakfast		


augment "/config:modules/config:module/config:state" {
        case kitchen-service-impl {
            when "/config:modules/config:module/config:type = 'kitchen-service-impl'";

            rpcx:rpc-context-instance "make-scrambled-with-wheat-rpc";
        }
    }

    identity make-scrambled-with-wheat-rpc;

    rpc make-scrambled-with-wheat  {
        description
          "Shortcut JMX call to make breakfast with scrambled eggs and wheat toast for testing.";

        input {
            uses rpcx:rpc-context-ref {
                refine context-instance {
                    rpcx:rpc-context-instance make-scrambled-with-wheat-rpc;
                }
            }
        }
        output {
            leaf result {
                type boolean;
            }
        }
    }
---
	   final KitchenServiceRuntimeRegistration runtimeReg =
									 getRootRuntimeBeanRegistratorWrapper().register( kitchenService );
	   ...
	   final class AutoCloseableKitchenService implements AutoCloseable {
		   @Override
		   public void close() throws Exception {
			   ...
			   runtimeReg.close();            
		   }
	   }
	   ...    		
---
		@Override
		public Boolean makeScrambledWithWheat() {
			try {
				// This call has to block since we must return a result to the JMX client.
				RpcResult<Void> result = makeBreakfast( EggsType.SCRAMBLED, WheatBread.class, 2 ).get();
				if( result.isSuccessful() ) {
					log.info( "makeBreakfast succeeded" );
				} else {
					log.warn( "makeBreakfast failed: " + result.getErrors() );
				}
	  
				return result.isSuccessful();
	  
			} catch( InterruptedException | ExecutionException e ) {
				log.warn( "An error occurred while maing breakfast: " + e );
			}
	  
			return Boolean.FALSE;
		}

---

## Part 6: Notifications

* Expand our example by adding notifications subscriptions from the toaster provider and 
* consumer that are routed through the MD-SAL.  

---
### yang model

		module toaster {
		   ... 
		   rpc restock-toaster {
			   description
				 "Restocks the toaster with the amount of bread specified.";
			   
		  
			   input {
				   leaf amountOfBreadToStock {
					   type uint32;
					   description
						 "Indicates the amount of bread to re-stock";
				   }
			   }
		   }
		   

		   notification toasterOutOfBread {
			 description
			   "Indicates that the toaster has run of out bread.";
		   }  // notification toasterOutOfStock
		   

		   notification toasterRestocked {
			 description
			   "Indicates that the toaster has run of out bread.";
			 leaf amountOfBread {
			   type uint32;
			   description
				 "Indicates the amount of bread that was re-stocked";
			 }
		   }  // notification toasterRestocked
		   
		 }  // module toaster
	
---
### impl model

	   augment "/config:modules/config:module/config:configuration" {
       case toaster-consumer-impl {
           when "/config:modules/config:module/config:type = 'toaster-consumer-impl'";
           ...
           

           container notification-service {
               uses config:service-ref {
                   refine type {
                       mandatory true;
                       config:required-identity mdsal:binding-notification-service;
                   }
               }
           }
       }

---
	   augment "/config:modules/config:module/config:configuration" {
		   case kitchen-service-impl {
			   when "/config:modules/config:module/config:type = 'kitchen-service-impl'";
			   ...
			   

			   container notification-service {
				   uses config:service-ref {
					   refine type {
						   mandatory true;
						   config:required-identity mdsal:binding-notification-service;
					   }
				   }
			   }
		   }
	   }
	   
      
---
### config model

		<snapshot>
		   <configuration>
			   
				   <modules xmlns="urn:opendaylight:params:xml:ns:yang:controller:config">
					   <module>
						   ...
						   <notification-service>
							   <type xmlns:binding="urn:opendaylight:params:xml:ns:yang:controller:md:sal:binding">
								   binding:binding-notification-service
							   </type>
							   <name>binding-notification-broker</name>
						   </notification-service>
					   </module>
				   </modules>
				   ...
			   
		   </configuration>
		   ...
		</snapshot>

	
---
### feature model

	
---
### service provider code model

		public class OpendaylightToaster implements ToasterService, ToasterProviderRuntimeMXBean, AutoCloseable, DataChangeListener {
		   ...
		   private NotificationProviderService notificationProvider;
		   ...
		   private final AtomicLong amountOfBreadInStock = new AtomicLong( 100 );
		   ...
		   public void setNotificationProvider(NotificationProviderService salService) {
			   this.notificationProvider = salService;
		   }
		   ...
		   

		   private void checkStatusAndMakeToast( final MakeToastInput input,
												 final SettableFuture<RpcResult<Void>> futureResult ) {
		  
			   ...  
			   final ListenableFuture<RpcResult<TransactionStatus>> commitFuture =
				   Futures.transform( readFuture, new AsyncFunction<Optional<DataObject>,
																		   RpcResult<TransactionStatus>>() {
		 
					   @Override
					   public ListenableFuture<RpcResult<TransactionStatus>> apply(
							   Optional<DataObject> toasterData ) throws Exception {
		 
						   ...
						   if( toasterStatus == ToasterStatus.Up ) {
		 
							  if( outOfBread() ) {
								  LOG.debug( "Toaster is out of bread" );
		 
								  return Futures.immediateFuture( Rpcs.<TransactionStatus>getRpcResult(
											  false, null, makeToasterOutOfBreadError() ) );
							  }
		 
							  ...
						  }
						  ...
					  }
			  } );
			  ...  
		  }
		  ...
		   /**
			* RestConf RPC call implemented from the ToasterService interface.
			* Restocks the bread for the toaster and sends a ToasterRestocked notification.
			*/
		   @Override
		   public Future<RpcResult<java.lang.Void>> restockToaster(RestockToasterInput input) {
			   LOG.info( "restockToaster: " + input );
			   
			   amountOfBreadInStock.set( input.getAmountOfBreadToStock() );
			   
			   if( amountOfBreadInStock.get() > 0 ) {
				   ToasterRestocked reStockedNotification =
					   new ToasterRestockedBuilder().setAmountOfBread( input.getAmountOfBreadToStock() ).build();
				   notificationProvider.publish( reStockedNotification );
			   }
			   
			   return Futures.immediateFuture(Rpcs.<Void> getRpcResult(true, Collections.<RpcError> emptySet()));
		   }
		   ...
		   private boolean outOfBread()
		   {
			   return amountOfBreadInStock.get() == 0;
		   }
		   

		   private class MakeToastTask implements Callable<Void> {
			   ...
			   @Override
			   public Void call() throws InterruptedException {
				   ...
				   amountOfBreadInStock.getAndDecrement();
				   if( outOfBread() ) {
					   LOG.info( "Toaster is out of bread!" );
					   
					   notificationProvider.publish( new ToasterOutOfBreadBuilder().build() );
				   }
				   ...
			   }
		   }

---
### service consumer code model

    public java.lang.AutoCloseable createInstance() {
       ...
       final Registration<NotificationListener> toasterListenerReg =
               getNotificationServiceDependency().registerNotificationListener( kitchenService );
       

       final KitchenServiceRuntimeRegistration runtimeReg =
               getRootRuntimeBeanRegistratorWrapper().register( kitchenService );
       

       final class AutoCloseableKitchenService implements KitchenService, AutoCloseable  {
           @Override
           public void close() throws Exception {
               toasterListenerReg.close();
               runtimeReg.close();
               log.info("Toaster consumer (instance {}) torn down.", this);
           }
           ...
       }
       ...
    }
---   


		public class KitchenServiceImpl implements KitchenService, KitchenServiceRuntimeMXBean, ToasterListener {
		   ...
		   private volatile boolean toasterOutOfBread;
		   

		   private Future<RpcResult<Void>> makeToast( Class<? extends ToastType> toastType,
													  int toastDoneness ) {
		 
			   if( toasterOutOfBread )
			   {
				   log.info( "We're out of toast but we can make eggs" );
				   return Futures.immediateFuture( Rpcs.<Void> getRpcResult( true,
							  Arrays.asList( RpcErrors.getRpcError( "", "partial-operation", null,
												 ErrorSeverity.WARNING,
												 "Toaster is out of bread but we can make you eggs",
												 ErrorType.APPLICATION, null ) ) ) );
			   }
		 
			   // Access the ToasterService to make the toast.
		 
			   MakeToastInput toastInput = new MakeToastInputBuilder()
				   .setToasterDoneness( (long) toastDoneness )
				   .setToasterToastType( toastType )
				   .build();
		 
			   return toaster.makeToast( toastInput );
		   }
		   ...
		   /**
			* Implemented from the ToasterListener interface.
			*/
		   @Override
		   public void onToasterOutOfBread( ToasterOutOfBread notification ) {
			   log.info( "ToasterOutOfBread notification" );
			   toasterOutOfBread = true;
		   }
		   

		   /**
			* Implemented from the ToasterListener interface.
			*/
		   @Override
		   public void onToasterRestocked( ToasterRestocked notification ) {
			   log.info( "ToasterRestocked notification - amountOfBread: " + notification.getAmountOfBread() );
			   toasterOutOfBread = false;
		   }
		}
---
	 
---
### test model

	HTTP Method => POST
	URL => http://localhost:8080/restconf/operations/toaster:restock-toaster 
	Header => Content-Type: application/yang.data+json  
	Body =>  
	{
	  "input" :
	  {
		 "toaster:amountOfBreadToStock" : "3"
	  }
	}	

---

    Open JConsole
    Navigate to the MBeans tab
    Expand the org.opendaylight.controller->RuntimeBean->kitchen-service-impl->kitchen-service-imp->Operations node.
    Click the makeScrambledWithWheat button 3 times.

    After the 3rd breakfast, the toaster will be out of bread and send the toasterOutOfBread notification to the kitchen service. You should see this message in the controller log/console:

       KitchenServiceImpl - ToasterOutOfBread notification

    Click the makeScrambledWithWheat button again - the result should be true and you should see this message in the log:

      KitchenServiceImpl - We're out of toast but we can make eggs

    Invoke restock-toaster again - you should see this message in the log:

      KitchenServiceImpl - ToasterRestocked notification - amountOfBread: 3


---
