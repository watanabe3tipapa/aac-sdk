## Core API

### Overview

The module contains the Engine base classes and the abstract platform interfaces that can be utilized by the platform and/or other modules. It also provides an easy way to integrate the Alexa Auto SDK into an application or a framework. This involves configuring and creating an instance of `aace::core::Engine`, overriding default platform implementation classes, and registering the custom interface handlers with the instantiated Engine.

### Creating the Engine

You create an instance of the Engine by calling the static function `aace::core::Engine::create()`.

    std::shared_ptr<aace::core::Engine> engine = aace::core::Engine::create();

### Configuring the Engine

Before the Engine can be started, you must configure it using the required `aace::core::config::EngineConfiguration` object(s) for the services you will be using. The SDK provides classes for reading the configuration data from a JSON file, as well as programmatically configuring the services.

    aace::core::config::ConfigurationFile::create("AlexaClientSDKConfig.json");

The class `aace::core::config::ConfigurationFile` creates an Engine configuration object from a file:

(example `.json` config file)

```
{
    "deviceInfo": {
        "deviceSerialNumber": "<DEVICE_SERIAL_NUMBER>",
        "clientId": "<CLIENT_ID>",
        "productId": "<PRODUCT_ID>"
    },
    "certifiedSender": {
        "databaseFilePath": "<SQLITE_DATABASE_FILE_PATH>"
    },
    "alertsCapabilityAgent": {
        "databaseFilePath": "<SQLITE_DATABASE_FILE_PATH>"
    },
    "settings": {
        "databaseFilePath": "<SQLITE_DATABASE_FILE_PATH>",
        "defaultAVSClientSettings": {
        "locale": "en-US"
        }
    }
}
```

The JSON file can contain all of the configuration data for the Engine or optionally it can be broken up into several files with multiple `aace::core::config::ConfigurationFile` objects.

Another way to specify the configuration data is programmatically by using the configuration factory methods classes provided in the library. For example, you can configure the `alertsCapabilityAgent` settings by instantiating a configuration object with the following method:

    auto alertsConfig = aace::alexa::config::AlexaConfiguration::createAlertsConfig( "<SQLITE_DATABASE_FILE_PATH>" );

After you have created your configuration object(s) you should call the Engine's `configure()` function, passing in the configuration object(s).

**NOTE:** The Engine's `configure()` method can only be called once and must be called before registering any platform interfaces or starting the Engine.

    engine->configure( config );

OR

    engine->configure( { deviceInfoConfig, alertsConfig, ... } );

### Registering Platform Interface Handlers

The Engine class provides two methods for registering platform interface handlers, allowing the developer to register one or more interfaces at a time for convenience.

    class MyInterface : public SpeechRecognizer {
    ...

    engine->registerPlatformInterface( std::make_shared<MyInterface>() );

    OR

    std::shared_ptr::MyInterface1 myInterface1 = std::make_shared<MyInterface1>();
    std::shared_ptr::MyInterface2 myInterface2 = std::make_shared<MyInterface2>();
    engine->registerPlatformInterface({ myInterface1, myInterface2 });

Details about extending the default platform interfaces are covered later in this document. See the section [**Extending the Default Platform Implementation**](#extending-the-default-platform-implementation) for more information.

### Starting the Engine

After the Engine has been created and configured, and all of the platform interfaces have been registered, the `Engine::start()` method must be called:

    engine->start();

### Extending the Default Platform Implementation

The default Alexa Auto SDK platform implementation can be extended by overriding the various classes in the library that, when registered with the Engine, allow you to interact with Amazon services.

### Implementing LocationProvider

The Engine provides a callback for implementing location requests from Alexa and other modules and a Location type definition. This is optional and dependent on the platform implementation.

To implement a custom LocationService handler to provide location using the default Engine LocationProvider class the `aace::location::LocationProvider` class should be extended:

    #include <AACE/Location/LocationProvider.h>

    class MyLocationProvider : public aace::location::LocationProvider {

      Location getLocation() override {
        // get platform location
        return m_platformLocation;
      }
      ...

      m_platformLocation = aace::location::Location(...);
    };

    //engine config
    engine->registerPlatformInterface( std::make_shared<MyLocationProvider>());

### Implementing NetworkInfoProvider

The Engine provides callbacks for implementing network information requests and informing the Engine of network changes. This is optional and dependent on the platform implementation.

To implement a custom NetworkInfoProvider handler to provide network info using the default Engine NetworkInfoProvider class the `aace::network::NetworkInfoProvider` class should be extended:

    #include <AACE/Network/NetworkInfoProvider.h>

    class MyNetworkInfoProvider : public aace::network::NetworkInfoProvider {

      NetworkStatus getNetworkStatus() override {
        // get platform network status
        return platformStatus;
      }
      int getWifiSignalStrength() override{
        //get current network RSSI
        return platformRSSI;
      }
      ...

      void MyNetworkStatusChangedHandler(...){
        //provide platform network status inform
        int rrsi;
        NetworkStatus status;
        networkStatusChanged( status, rssi);
      }
      ...
    };

    //engine config
    engine->registerPlatformInterface( std::make_shared<MyNetworkInfoProvider>());

### Implementing Log Events

The Engine provides a callback for implementing log events from the AVS SDK. This is entirely optional for the platform implementation.

To implement a custom log event handler for logging events from AVS using the default engine Logger class the `aace::logging::Logger` class should be extended:

    #include <AACE/Logger/Logger.h>

    class MyLogger : public aace::logger::Logger {

      void logEvent(aace::logger::Logger::Level level, std::chrono::system_clock::time_point time, const std::string& source, const std::string& message) override {
        //handle the log message
      };
      ...

    };

    //engine config
    engine->registerPlatformInterface( std::make_shared<MyLogger>());
