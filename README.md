# JBoss EAP 6.3 Beta - WebSocket Demo

This is a demonstration on how to get WebSockets running on JBoss EAP 6.3 Beta. The full source code for the demo is
available as part of the git project, but in this file contains documentation on how to create a demo like this without
clone it on github.




## Steps to create
In this demo we will create a simple websocket demo that will echo your name in a few short steps.

1. Download JBoss EAP 6.3 Beta (or later) from [http://www.jboss.org/products](http://www.jboss.org/products). In this example we will use the zip version and not the installer.

1. Install JBoss by unzipping in the directory of choice which is listed as `<install-dir>`.```
    $ unzip -d <install-dir> jboss-eap-6.3.0.Beta.zip```

2. Start JBoss in standalone mode by running the following commands.```
    $ cd <install-dir>/jboss-eap-6.3/bin
    $ ./standalone.sh```

3. Open IDE of choice and create a maven project.

4. Edit pom.xml add change package type to `war`.```
    <packaging>war</packaging>```

4. Edit pom.xml add dependencies to ```
        <dependencies>
            <dependency>
                <groupId>org.jboss.spec</groupId>
                <artifactId>jboss-javaee-web-6.0</artifactId>
                <version>3.0.2.Final</version>
                <type>pom</type>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.jboss.spec.javax.websocket</groupId>
                <artifactId>jboss-websocket-api_1.0_spec</artifactId>
                <version>1.0.0.Final</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>```

5. Create a java class named `org.jboss.as.demo.wsdemo.endpoints.HelloEndpoint`

5. Edit the java class to look like this: ```
package org.jboss.as.demos.wsdemo.endpoints;

import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint("/websocket/hello")
public class HelloEndpoint {

    @OnMessage
    public String sayHello(String name) {
        System.out.println("Say hello to '" + name + "'");
        return ("Hello " + name);
    }

    @OnOpen
    public void helloOnOpen(Session session) {
        System.out.println("WebSocket opened: " + session.getId());
    }

    @OnClose
    public void helloOnClose(CloseReason reason) {
        System.out.println("Closing a WebSocket due to " + reason.getReasonPhrase());
    }

}```

5. Make sure that the project builds correctly by running `mvn clean package`

