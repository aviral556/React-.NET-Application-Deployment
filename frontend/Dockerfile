# .NET Dockerfile
FROM mcr.microsoft.comdotnetsdk7.0 AS build
WORKDIR app
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.comdotnetaspnet7.0
WORKDIR app
COPY --from=build appout .
ENTRYPOINT [dotnet, YourApp.dll]
