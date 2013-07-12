# OpenShift Cartridge for Jetty WebSocket 

Create an app with a downloaded cartridge: http://cartreflect-claytondev.rhcloud.com/reflect?github=AtosWorldline/openshift-cartridge-jetty-websocket

The cartridge will run :
* `mvn install -Popenshift -DskipTests=true` to build your application
* `mvn -Popenshift jetty:run` to start it

Logs are written in `$OPENSHIFT_JETTY_DIR/logs/jetty.log`

A `pom.xml` with `maven-jetty-plugin` is expected at the root of your repo. If your project is more complex, feel free to fork and adapt `bin/control` to your needs.

There is an [example application](https://github.com/AtosWorldline/openshift-cartridge-jetty-websocket/tree/master/template), that may help you to understand your to configure your project to make it works on openshift.

In your pom.xml, create an [openshift profile](https://github.com/AtosWorldline/openshift-cartridge-jetty-websocket/blob/master/template/pom.xml#L11) where you define jetty.port and jetty.host variable:

    <profiles>
      <profile>
        <id>openshift</id>
        <properties>
          <jetty.port>${env.OPENSHIFT_JETTY_PORT}</jetty.port>
          <jetty.host>${env.OPENSHIFT_JETTY_IP}</jetty.host>
        </properties>
      </profile>
      <profile>
        <id>default</id>
        <activation>
          <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
          <jetty.port>8080</jetty.port>
          <jetty.host>0.0.0.0</jetty.host>
        </properties>
      </profile>
    </profiles>

Then create a [connector in the `jetty-maven-plugin` configuration](https://github.com/AtosWorldline/openshift-cartridge-jetty-websocket/blob/master/template/pom.xml#L41), to configure the `port` and the `host`

    <plugin>
      <!-- This plugin is needed for the servlet example -->
      <groupId>org.mortbay.jetty</groupId>
      <artifactId>jetty-maven-plugin</artifactId>
      <version>8.1.0.v20120127</version>
      <configuration>
        <scanIntervalSeconds>10</scanIntervalSeconds>
        <connectors>
          <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
            <port>${jetty.port}</port>
            <host>${jetty.host}</host>
            <maxIdleTime>60000</maxIdleTime>
          </connector>
        </connectors>
        <webAppConfig>
          <contextPath>/</contextPath>
          <descriptor>${basedir}/src/main/webapp/web.xml</descriptor>
        </webAppConfig>
      </configuration>
    </plugin>

Use 8000 port or 8443 for wss ! Web traffic works on the 80 and 443 port on OpenShift but WebSocket only works on the 8000 and 8443 port.

So you will have to adapt your client like this ([JavaScript example](https://github.com/AtosWorldline/openshift-cartridge-jetty-websocket/blob/master/template/src/main/webapp/index.jsp#L44)): 

    var openshiftWebSocketPort = 8000; // Or use 8443 for wss
    var wsUri = "ws://" + window.location.hostname + ":" + openshiftWebSocketPort + "/ws";
    war ws = new WebSocket(wsUri);
    ...

