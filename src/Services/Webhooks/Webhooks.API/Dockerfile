FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src

# It's important to keep lines from here down to "COPY . ." identical in all Dockerfiles
# to take advantage of Docker's build cache, to speed up local container builds
COPY "eShopOnContainers-ServicesAndWebApps.sln" "eShopOnContainers-ServicesAndWebApps.sln"

COPY "ApiGateways/Mobile.Bff.Shopping/aggregator/Mobile.Shopping.HttpAggregator.csproj" "ApiGateways/Mobile.Bff.Shopping/aggregator/Mobile.Shopping.HttpAggregator.csproj"
COPY "ApiGateways/Web.Bff.Shopping/aggregator/Web.Shopping.HttpAggregator.csproj" "ApiGateways/Web.Bff.Shopping/aggregator/Web.Shopping.HttpAggregator.csproj"
COPY "BuildingBlocks/EventBus/EventBus/EventBus.csproj" "BuildingBlocks/EventBus/EventBus/EventBus.csproj"
COPY "BuildingBlocks/EventBus/EventBus.Tests/EventBus.Tests.csproj" "BuildingBlocks/EventBus/EventBus.Tests/EventBus.Tests.csproj"
COPY "BuildingBlocks/EventBus/EventBusRabbitMQ/EventBusRabbitMQ.csproj" "BuildingBlocks/EventBus/EventBusRabbitMQ/EventBusRabbitMQ.csproj"
COPY "BuildingBlocks/EventBus/EventBusServiceBus/EventBusServiceBus.csproj" "BuildingBlocks/EventBus/EventBusServiceBus/EventBusServiceBus.csproj"
COPY "BuildingBlocks/EventBus/IntegrationEventLogEF/IntegrationEventLogEF.csproj" "BuildingBlocks/EventBus/IntegrationEventLogEF/IntegrationEventLogEF.csproj"
COPY "BuildingBlocks/WebHostCustomization/WebHost.Customization/WebHost.Customization.csproj" "BuildingBlocks/WebHostCustomization/WebHost.Customization/WebHost.Customization.csproj"
COPY "Services/Basket/Basket.API/Basket.API.csproj" "Services/Basket/Basket.API/Basket.API.csproj"
COPY "Services/Basket/Basket.FunctionalTests/Basket.FunctionalTests.csproj" "Services/Basket/Basket.FunctionalTests/Basket.FunctionalTests.csproj"
COPY "Services/Basket/Basket.UnitTests/Basket.UnitTests.csproj" "Services/Basket/Basket.UnitTests/Basket.UnitTests.csproj"
COPY "Services/Catalog/Catalog.API/Catalog.API.csproj" "Services/Catalog/Catalog.API/Catalog.API.csproj"
COPY "Services/Catalog/Catalog.FunctionalTests/Catalog.FunctionalTests.csproj" "Services/Catalog/Catalog.FunctionalTests/Catalog.FunctionalTests.csproj"
COPY "Services/Catalog/Catalog.UnitTests/Catalog.UnitTests.csproj" "Services/Catalog/Catalog.UnitTests/Catalog.UnitTests.csproj"
COPY "Services/Identity/Identity.API/Identity.API.csproj" "Services/Identity/Identity.API/Identity.API.csproj"
COPY "Services/Ordering/Ordering.API/Ordering.API.csproj" "Services/Ordering/Ordering.API/Ordering.API.csproj"
COPY "Services/Ordering/Ordering.BackgroundTasks/Ordering.BackgroundTasks.csproj" "Services/Ordering/Ordering.BackgroundTasks/Ordering.BackgroundTasks.csproj"
COPY "Services/Ordering/Ordering.Domain/Ordering.Domain.csproj" "Services/Ordering/Ordering.Domain/Ordering.Domain.csproj"
COPY "Services/Ordering/Ordering.FunctionalTests/Ordering.FunctionalTests.csproj" "Services/Ordering/Ordering.FunctionalTests/Ordering.FunctionalTests.csproj"
COPY "Services/Ordering/Ordering.Infrastructure/Ordering.Infrastructure.csproj" "Services/Ordering/Ordering.Infrastructure/Ordering.Infrastructure.csproj"
COPY "Services/Ordering/Ordering.SignalrHub/Ordering.SignalrHub.csproj" "Services/Ordering/Ordering.SignalrHub/Ordering.SignalrHub.csproj"
COPY "Services/Ordering/Ordering.UnitTests/Ordering.UnitTests.csproj" "Services/Ordering/Ordering.UnitTests/Ordering.UnitTests.csproj"
COPY "Services/Payment/Payment.API/Payment.API.csproj" "Services/Payment/Payment.API/Payment.API.csproj"
COPY "Services/Services.Common/Services.Common.csproj" "Services/Services.Common/Services.Common.csproj"
COPY "Services/Webhooks/Webhooks.API/Webhooks.API.csproj" "Services/Webhooks/Webhooks.API/Webhooks.API.csproj"
COPY "Tests/Services/Application.FunctionalTests/Application.FunctionalTests.csproj" "Tests/Services/Application.FunctionalTests/Application.FunctionalTests.csproj"
COPY "Web/WebhookClient/WebhookClient.csproj" "Web/WebhookClient/WebhookClient.csproj"
COPY "Web/WebMVC/WebMVC.csproj" "Web/WebMVC/WebMVC.csproj"
COPY "Web/WebSPA/WebSPA.csproj" "Web/WebSPA/WebSPA.csproj"
COPY "Web/WebStatus/WebStatus.csproj" "Web/WebStatus/WebStatus.csproj"

COPY "docker-compose.dcproj" "docker-compose.dcproj"

COPY "Directory.Packages.props" "Directory.Packages.props"
COPY "NuGet.config" "NuGet.config"

RUN dotnet restore "eShopOnContainers-ServicesAndWebApps.sln"

COPY . .
WORKDIR /src/Services/Webhooks/Webhooks.API
RUN dotnet publish --no-restore -c Release -o /app

FROM build AS publish

FROM base AS final

# Opentelemetry DOTNET SPLUNK IMPL

# Add the Splunk Distribution of OpenTelemetry .NET
ARG SPLUNK_OTEL_DOTNET_VERSION=v0.1.0-beta.1
ADD https://github.com/signalfx/splunk-otel-dotnet/releases/download/${SPLUNK_OTEL_DOTNET_VERSION}/splunk-opentelemetry-dotnet-linux-glibc.zip ./splunk-otel-dotnet.zip

RUN apt update
RUN apt -y install zip

ARG INSTALL_DIR=/opt/splunk/otel/dotnet
RUN mkdir -p ${INSTALL_DIR}
RUN unzip splunk-otel-dotnet.zip -d ${INSTALL_DIR}
RUN rm splunk-otel-dotnet.zip
RUN mkdir -p /var/log/opentelemetry/dotnet

ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER={918728DD-259F-4A6A-AC2B-B85E1B658318}
ENV CORECLR_PROFILER_PATH=${INSTALL_DIR}/linux-64/OpenTelemetry.AutoInstrumentation.Native.so
ENV DOTNET_ADDITIONAL_DEPS=${INSTALL_DIR}/AdditionalDeps
ENV DOTNET_SHARED_STORE=${INSTALL_DIR}/store
ENV DOTNET_STARTUP_HOOKS=${INSTALL_DIR}/net/OpenTelemetry.AutoInstrumentation.StartupHook.dll
ENV OTEL_DOTNET_AUTO_INTEGRATIONS_FILE=${INSTALL_DIR}/integrations.json
ENV OTEL_DOTNET_AUTO_HOME=${INSTALL_DIR}

ENV OTEL_SERVICE_NAME=WebhooksAPI
ENV OTEL_RESOURCE_ATTRIBUTES=deployment.environment=dd-otel-dotnet,service.version=1.0.0
ENV OTEL_EXPORTER_OTLP_ENDPOINT=http://54.219.132.60:4318


WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "Webhooks.API.dll"]
