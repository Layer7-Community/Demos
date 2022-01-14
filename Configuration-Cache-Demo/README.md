# Configuration Cache Demo
A demonstration of pulling configuration from an external source and pushing to cache.

# Overview
There have been several occasions where customers want to control configuration parameters for Gateway policies in an external repository, such as a database or a service. This demonstration illustrates one way of doing this by storing the configuration for a simple passthrough test service in an XML document presented by a service, which is periodically accessed by a scheduled task on the Gateway that parses the XML and stores the parameters in the local cache. The passthrough test service reads the values from the cache and echoes the parameters back to the requestor.

The configuration XML describes two endpoints and is hardcoded in the Demo Configuration Service at /ConfigurationCacheDemo/configuration:

    <configuration>
      <service path="/ConfigurationCacheDemo/test">
        <parameter name="url">http://www.foo123.com/test</parameter>
        <parameter name="connectTimeout">3500</parameter>
        <parameter name="readTimeout">10000</parameter>
      </service>
      <service path="/ConfigurationCacheDemo/auth">
        <parameter name="url">http://www.foo123.com/auth</parameter>
        <parameter name="connectTimeout">2500</parameter>
        <parameter name="readTimeout">30000</parameter>
      </service>
    </configuration>

The Load Configuration Cache scheduled task policy makes a routing request to /ConfigurationCacheDemo/configuration, parses the XML and loads the parameters into the cache.

Finally, the Cached Configuration Demo Service service at /ConfigurationCacheDemo/* reads the parameters from the cache for the service (assuming they are found - if not it results in a 404 response) end returns a simple template listing the parameters.

# Installation
Download Configuration-Cache-Demo.bundle and deploy using the GMU (assumes connection details are in &lt;gateway&gt;.properties):

    # ./GatewayMigrationUtility.sh migrateIn -z <gateway>.properties -b Configuration-Cache-Demo.bundle -r Configuration-Cache-Demo.result

You can also deploy directly to restman using curl:

    # curl -k -X PUT -d @Configuration-Cache-Demo.bundle -H "Content-Type: application/xml" -u admin:<password> -o Configuration-Cache-Demo.result 'https://<gateway>:8443/restman/1.0/bundle?test=false&activate=true'

Finally, you can deploy the bundle using the bootstrap bundle provisioning feature:

    # mkdir -p /opt/SecureSpan/Gateway/node/default/etc/bootstrap/bundle
    copy the bundle file to /opt/SecureSpan/Gateway/node/default/etc/bootstrap/bundle
    # chmod -R 775 /opt/SecureSpan/Gateway/node/default/etc/bootstrap
    # systemctl restart ssg

Note that if you are already connected to the Gateway with the Policy Manager, you may need to disconnect and reconnect to load the Routing Logic encapsulated assertion properly.

# Testing
Point a browser or curl to the passthrough test service as http://&lt;gateway&gt;:8080/ConfigurationCacheDemo/test:

    $ curl http://ssg100dest.l7tech.com:8080/ConfigurationCacheDemo/test
    Configuration for /ConfigurationCacheDemo/test:
      - url = http://www.foo123.com/test
      - readTimeout = 10000
      - connectTimeout = 3500
    $

Point a browser or curl to the passthrough test service as http://&lt;gateway&gt;:8080/ConfigurationCacheDemo/auth:

    $ curl http://ssg100dest.l7tech.com:8080/ConfigurationCacheDemo/auth
    Configuration for /ConfigurationCacheDemo/auth:
      - url = http://www.foo123.com/test
      - readTimeout = 30000
      - connectTimeout = 2500
    $

Point a browser or curl to the passthrough test service as http://&lt;gateway&gt;:8080/ConfigurationCacheDemo/bogus:

    $ curl http://ssg100dest.l7tech.com:8080/ConfigurationCacheDemo/bogus
    Not Found
    $

Open the Demo Configuration Service policy and edit the template to either add a new service to the XML or alter existing parameter values. Wait at most ten seconds for the values to be loaded by the scheduled task then access the passthrough test service again to see the updated configuration.

