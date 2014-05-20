# JBoss EAP 6.3 Beta - WebSocket Demo

This is a demonstration on how to get WebSockets running on JBoss EAP 6.3 Beta. The full source code for the demo is
available as part of the git project, but in this file contains documentation on how to create a demo like this without
clone it on github.




## Steps to create and build the application
In this demo we will create a simple websocket demo that will echo your name in a few short steps.

1. Download JBoss EAP 6.3 Beta (or later) from [http://www.jboss.org/products/eap](http://www.jboss.org/products/eap). In this example we will use the zip version and not the installer.

1. Install JBoss by unzipping in the directory of choice which is listed as `<install-dir>`.

        $ unzip -d <install-dir> jboss-eap-6.3.0.Beta.zip

2. Start JBoss in standalone mode by running the following commands.

        $ cd <install-dir>/jboss-eap-6.3/bin
        $ ./standalone.sh

3. Open IDE of choice and create a maven project.

4. Edit pom.xml add change package type to `war`.

        <packaging>war</packaging>

4. Edit pom.xml add dependencies to

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
        </dependencies>

4. Edit pom.xml to add plugins for compile and war like this

        <build>
            <finalName>${project.artifactId}</finalName>
            <pluginManagement>

                <plugins>
                    <!-- Compiler plugin enforces Java 1.6 compatibility and activates
                  annotation processors -->
                    <plugin>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>2.3.1</version>
                        <configuration>
                            <source>1.6</source>
                            <target>1.6</target>
                        </configuration>
                    </plugin>
                    <plugin>
                        <artifactId>maven-war-plugin</artifactId>
                        <version>2.1.1</version>
                        <configuration>
                            <failOnMissingWebXml>false</failOnMissingWebXml>
                        </configuration>
                    </plugin>
                </plugins>
            </pluginManagement>
        </build>


5. Create a java class named `org.jboss.as.demos.wsdemo.endpoints.HelloEndpoint`

5. Edit the java class to look like this:

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

        }

5. Make sure that the project builds correctly by running `mvn clean package`

6. Add the following HTML5 page with java-script to call the websocket endpoint

        <!DOCTYPE html>
        <html lang="en">
        <head>
            <title>WebSocket Demo</title>
            <meta charset="utf-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <script type="text/javascript">
                    var websocket = null;

                    function connect() {
                        var wsURI = 'ws://' + window.location.host + '/wsdemo/websocket/hello';
                        websocket = new WebSocket(wsURI);

                        websocket.onopen = function() {
                            displayStatus('Open');
                            document.getElementById('sayHello').disabled = false;
                            displayMessage('Connection is now open. Type a name and click Say Hello to send a message.');
                        };
                        websocket.onmessage = function(event) {
                            // log the event
                            displayMessage('The response was received! ' + event.data, 'success');
                        };
                        websocket.onerror = function(event) {
                            // log the event
                            displayMessage('Error! ' + event.data, 'error');
                        };
                        websocket.onclose = function() {
                            displayStatus('Closed');
                            displayMessage('The connection was closed or timed out. Please click the Open Connection button to reconnect.');
                            document.getElementById('sayHello').disabled = true;
                        };
                    }

                    function disconnect() {
                        if (websocket !== null) {
                            websocket.close();
                            websocket = null;
                        }
                        message.setAttribute("class", "message");
                        message.value = 'WebSocket closed.';
                        // log the event
                    }

                    function sendMessage() {
                        if (websocket !== null) {
                            var content = document.getElementById('name').value;
                            websocket.send(content);
                        } else {
                            displayMessage('WebSocket connection is not established. Please click the Open Connection button.', 'error');
                        }
                    }

                    function displayMessage(data, style) {
                        var message = document.getElementById('hellomessage');
                        message.setAttribute("class", style);
                        message.value = data;
                    }

                    function displayStatus(status) {
                        var currentStatus = document.getElementById('currentstatus');
                        currentStatus.value = status;
                    }

            </script>
            <link href="css/bootstrap.min.css" rel="stylesheet">

        </head>
        <body>
        <div class="container">
            <h1>Welcome to JBoss!</h1>
            <div>This is a simple example of a WebSocket implementation.</div>
            <div id="connect-container">
                <div>
                    <fieldset>
                        <legend>Connect or disconnect using WebSocket :</legend>
                        <input type="button" id="connect" onclick="connect();" value="Open Connection" />
                        <input type="button" id="disconnect" onclick="disconnect();" value="Close Connection" />
                    </fieldset>
                </div>
                <div>
                    <fieldset>
                        <legend>Type your name below. then click the `Say Hello` button :</legend>
                        <input id="name" type="text" size="40" style="width: 40%"/>
                        <input type="button" id="sayHello" onclick="sendMessage();" value="Say Hello" disabled="disabled"/>
                    </fieldset>
                </div>
                <div>Current WebSocket Connection Status: <output id="currentstatus" class="message">Closed</output></div>
                <div>
                    <output id="hellomessage" />
                </div>
            </div>
        </div>
        <script src="js/bootstrap.min.js"></script>
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.0/jquery.min.js"></script>
        </body>
        </html>

## Steps to configure JBoss EAP and deploy the application

1. To enable JBoss EAP for websockets we need to change the http connector to user either NIO or Apache Portable Runtime
(APR), but for this demo we will use NIO. To change this we can execute the following CLI command using jboss_cli.sh|bat

        /subsystem=web/connector=http/:write-attribute(name=protocol,value=org.apache.coyote.http11.Http11NioProtocol)
We also need to reload the configuration by running

        :reload
For local development environment we can use the jboss-as maven plugin to configure JBoss by adding the following to the pom.xml

        <plugin>
            <groupId>org.jboss.as.plugins</groupId>
            <artifactId>jboss-as-maven-plugin</artifactId>
            <version>7.4.Final</version>
            <configuration>
                <execute-commands>
                    <batch>true</batch>
                    <commands>
                        <command>/subsystem=web/connector=http/:write-attribute(name=protocol,value=org.apache.coyote.http11.Http11NioProtocol)</command>
                        <command>:reload</command>
                    </commands>
                </execute-commands>
            </configuration>
        </plugin>

Then we use the follwing commnad to apply the configuration (JBoss EAP needs to be running)

        mvn jboss-as:execute-commands

2. Now we can deploy the application using CLI, GUI or maven. For this demo we will use the jboss-as maven plugin.

        mvn package jboss-as:deploy

3. Open a browser to [http://localhost:8080/wsdemo/index.html](http://localhost:8080/wsdemo/index.html)
