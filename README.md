# DEPI_EUI_DOCKER_SIMPLE_DOTNET_APP
# Create a simple ASP.NET app, build an image from the app then push it to dockerhub and run a container from the app

references: 
https://www.atlantic.net/dedicated-server-hosting/how-to-deploy-net-core-applications-on-ubuntu-22-04/
https://hbayraktar.medium.com/how-to-dockerize-a-net-8-asp-net-core-web-application-b15f63246535


**Create a simple ASP.NET app**

sudo mkdir /var/www/spot_ubuntu
sudo chmod -R 777 /var/www/spot_ubuntu
sudo apt install snap
sudo snap install dotnet-sdk --classic --channel=7.0
sudo ln -s /snap/bin/dotnet-sdk.dotnet /usr/bin/dotnet
dotnet --version ---> to check

dotnet new webapp -o MyApp --no-https ---> create a new app & create a new directory "MyApp"

![image](https://github.com/user-attachments/assets/d5f4d3cf-3abc-46dc-adf1-ff693895b163)

cd MyApp/ ---> go to the directory
dotnet build

![image](https://github.com/user-attachments/assets/0f1880a9-97d9-47f3-9382-227c235a2820)

sudo dotnet publish -c Release -o /var/www/ --runtime linux-x64

**Create a Systemd File for .NET Application**

sudo vim /etc/systemd/system/app.service
```
[Unit]
Description=My .NET Core Application

[Service]
WorkingDirectory=/var/www/
ExecStart=/usr/bin/dotnet /var/www/MyApp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=root
Environment=ASPNETCORE_ENVIRONMENT=Production

[Install]
WantedBy=multi-user.target
```
sudo systemctl daemon-reload
sudo systemctl start app
sudo systemctl status app

![image](https://github.com/user-attachments/assets/fd0d7faf-b3cc-4829-a89b-77abc15d14f5)


**Create an Image from the SAP.NET Application**

cd MyApp/ ---> in My App directory

vim Dockerfile

```
# Use .NET 8 SDK to build the application
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build

# Set the working directory
WORKDIR /app

# Copy the project files
COPY . .

# Restore and build the project
RUN dotnet restore
RUN dotnet publish -c Release -o out

# Final stage: run the application using the ASP.NET Core runtime
FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /app
COPY --from=build /app/out .

# Expose port 80 for the application
EXPOSE 80

# Set the application to listen on IPv4 only
ENV ASPNETCORE_URLS=http://0.0.0.0:80

# Run the application
ENTRYPOINT ["dotnet", "MyApp.dll"]
```
docker build -t mydotnetapp .

**Run a container from the SAP.NET Application Image**

sudo docker run -d -p 8080:80 --name dotnet mydotnetapp
![image](https://github.com/user-attachments/assets/674803fb-1e26-472c-9602-d88e5bb86af0)

**Push the SAP.NET Application Image to dockerhub local registry**

sudo docker login
sudo docker tag mydotnetapp ranahesham/mydotnetapp:v1
sudo docker push ranahesham/mydotnetapp:v1

![image](https://github.com/user-attachments/assets/47a933fe-e229-474e-8b70-e68e022e0fa3)



