FROM tomcat:9.0-jre11

# Remove default Tomcat webapps
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy the probe WAR file
COPY prob.war /usr/local/tomcat/webapps/probe.war

# Create directory for custom configuration
RUN mkdir -p /usr/local/tomcat/conf/Catalina/localhost

# Install openssl for TLS certificate generation
RUN apt-get update && apt-get install -y openssl && rm -rf /var/lib/apt/lists/*

# Generate self-signed certificate
RUN keytool -genkeypair \
    -alias tomcat \
    -keyalg RSA \
    -keysize 2048 \
    -keystore /usr/local/tomcat/conf/keystore.p12 \
    -storetype PKCS12 \
    -storepass changeit \
    -keypass changeit \
    -dname "CN=localhost, OU=Development, O=Company, L=City, ST=State, C=US" \
    -validity 3650

# Set proper permissions
RUN chmod 644 /usr/local/tomcat/conf/keystore.p12

# Expose ports
EXPOSE 8080 8443

# Start Tomcat
CMD ["catalina.sh", "run"]