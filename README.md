# Nexus Artifact Repository Manager

## Overview
Nexus Artifact Repository Manager is a powerful tool for managing artifacts such as JAR, WAR, TAR, ZIP, and other formats. Nexus supports both open-source and commercial versions and can serve as a private repository or proxy for public repositories like Maven Central or npm. This project demonstrates the setup and use of Nexus on a cloud server using Ubuntu WSL.

---

## Features
- **Private Repository Hosting**: Host and manage your own artifacts securely.
- **Proxy Repositories**: Cache artifacts from public repositories, reducing build times and external dependency.
- **Flexible REST API**: Integrate Nexus into your CI/CD pipelines and automate tasks.
- **Blob Storage Management**: Efficiently manage binary artifact storage.
- **Clean-Up Policies**: Ensure optimal storage usage by automatically removing outdated artifacts.

---

## Prerequisites
- **Java Version 8**
  ```bash
  sudo apt-get install openjdk-8-jdk
  ```
- **Ubuntu WSL or Equivalent Cloud Server Environment**

---

## Setup Steps

### 1. Install Nexus
1. Download Nexus:
   ```bash
   wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
   ```
2. Extract the archive:
   ```bash
   tar -zxvf latest-unix.tar.gz
   ```

### 2. Create Nexus User
1. Add a new user for Nexus:
   ```bash
   adduser nexus
   ```
2. Change ownership of Nexus files:
   ```bash
   chown -R nexus:nexus nexus-3.x.x-xx
   chown -R nexus:nexus sonatype-work
   ```
3. Configure Nexus to run as the new user:
   ```bash
   vim nexus-3.x.x-xx/bin/nexus.rc
   ```
   Set:
   ```bash
   run_as_user="nexus"
   ```

### 3. Start Nexus
1. Switch to the Nexus user:
   ```bash
   su - nexus
   ```
2. Start the Nexus service:
   ```bash
   /opt/nexus-3.x.x-xx/bin/nexus start
   ```
3. Confirm Nexus is running:
   ```bash
   ps aux | grep nexus
   netstat -lnpt
   ```

### 4. Access the Nexus UI
- Open your browser and navigate to: `http://<private-IP>:8081`
- Default Admin Credentials:
  - Username: `admin`
  - Password: Located in the `admin.password` file in the `sonatype-work` directory.

---

## Repository Types
1. **Proxy Repository**: Caches artifacts from remote repositories like Maven Central.
2. **Hosted Repository**: Stores company-owned artifacts or third-party dependencies.
3. **Group Repository**: Combines multiple repositories under one endpoint for simplified management.

---

## Configuring Artifact Deployment

### Gradle Project
1. Add the following to your `build.gradle` file:
   ```groovy
   apply plugin: "maven-publish"

   publishing {
       publications {
           maven(MavenPublication) {
               artifact("build/libs/artifact-practice-gradle-app-$version" + ".jar") {
                   extension 'jar'
               }
           }
       }

       repositories {
           maven {
               name "nexus"
               url "http://<private-IP>:8081/repository/maven-snapshots/"
               allowInsecureProtocol true
               credentials {
                   username project.repoUser
                   password project.repoPassword
               }
           }
       }
   }
   ```
2. Define credentials in `gradle.properties`:
   ```
   repoUser = yourUsername
   repoPassword = yourPassword
   ```
3. Publish the artifact:
   ```bash
   ./gradlew publish
   ```

### Maven Project
1. Add the following to your `pom.xml`:
   ```xml
   <distributionManagement>
       <snapshotRepository>
           <id>nexus-snapshot</id>
           <url>http://<private-IP>:8081/repository/maven-snapshots/</url>
       </snapshotRepository>
   </distributionManagement>

   <build>
       <plugins>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-deploy-plugin</artifactId>
               <version>2.8.2</version>
           </plugin>
       </plugins>
   </build>
   ```
2. Configure credentials in `~/.m2/settings.xml`:
   ```xml
   <settings>
       <servers>
           <server>
               <id>nexus-snapshot</id>
               <username>yourUsername</username>
               <password>yourPassword</password>
           </server>
       </servers>
   </settings>
   ```
3. Deploy the artifact:
   ```bash
   mvn deploy
   ```

---

## Using Nexus REST API
- **List Repositories**:
  ```bash
  curl -u <username> -X GET 'http://<private-IP>:8081/service/rest/v1/repositories'
  ```
- **List Components in a Repository**:
  ```bash
  curl -u <username> -X GET 'http://<private-IP>:8081/service/rest/v1/components?repository=<repository-name>'
  ```

---

## Clean-Up Policies
- **Soft Delete**: Marks artifacts for deletion.
- **Hard Delete**: Permanently removes artifacts using the "Admin Compact Blob Store" task.

---

## Conclusion
This project demonstrates the end-to-end process of setting up, configuring, and using Nexus as an artifact repository manager. From installing Nexus to configuring repositories and deploying artifacts, this guide provides a comprehensive overview for developers and DevOps professionals.


