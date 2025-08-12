# Sonarqube .NET

**Install sonarqube by docker**

```yml
services:
  sonarqube-db:
    image: postgres:15-alpine
    container_name: sonarqube-postgres
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonarqube
    volumes:
      - sonarqube_postgres_data:/var/lib/postgresql/data
    networks:
      - sonarqube-network
    restart: unless-stopped

  sonarqube:
    image: sonarqube:10.3-community
    container_name: sonarqube-server
    depends_on:
      - sonarqube-db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://sonarqube-db:5432/sonarqube
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
      SONAR_ES_BOOTSTRAP_CHECKS_DISABLE: true
    ports:
      - "9000:9000"
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_extensions:/opt/sonarqube/extensions
    networks:
      - sonarqube-network
    restart: unless-stopped
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

volumes:
  sonarqube_postgres_data:
  sonarqube_data:
  sonarqube_logs:
  sonarqube_extensions:

networks:
  sonarqube-network:
    driver: bridge
```

**Install tool dotnet-sonarscanner**

```sh
dotnet tool install --global dotnet-sonarscanner
```

**run-sonar.sh**

```sh
#!/bin/bash

# Remove any existing results directory to ensure a clean state
rm -rf ./TestResults

# Run tests with OpenCover format for SonarQube compatibility
dotnet test --collect:"XPlat Code Coverage" --results-directory ./TestResults -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover

# Begin SonarQube analysis with OpenCover format
dotnet sonarscanner begin /k:"kozy-api" \
    /d:sonar.host.url="http://localhost:9000" \
    /d:sonar.token="sqp_c7fd42b17716ff30ad1b0901913a671a8b2e12e5" \
    /d:sonar.scanner.scanAll=false \
    /d:sonar.cs.opencover.reportsPaths="TestResults/**/coverage.opencover.xml" \
    /d:sonar.exclusions="**/bin/**,**/obj/**,**/wwwroot/**,**/Migrations/**,**/Program.cs,**/Dockerfile" \
    /d:sonar.test.exclusions="**/bin/**,**/obj/**"

# Build the project
dotnet build --no-restore

# End SonarQube analysis
dotnet sonarscanner end /d:sonar.token="sqp_c7fd42b17716ff30ad1b0901913a671a8b2e12e5"
```
