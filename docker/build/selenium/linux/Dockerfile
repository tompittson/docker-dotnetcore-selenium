FROM mcr.microsoft.com/dotnet/core/sdk:2.2

RUN set -x \
 # Install dependencies
 && apt-get update && apt-get install -y --no-install-recommends \
    apt-transport-https \
    ca-certificates \
    curl \
    unzip \
    gnupg \
    hicolor-icon-theme \
    libcanberra-gtk* \
    libgl1-mesa-dri \
    libgl1-mesa-glx \
    libpango1.0-0 \
    libpulse0 \
    libv4l-0 \
    fonts-symbola \
 # Add Chrome signing key and source
 && curl -sSL https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
 && echo "deb [arch=amd64] https://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list \
 # Install Chrome
 && apt-get update && apt-get install -y --no-install-recommends \
    google-chrome-stable \
 # Download the google-talkplugin
 && curl -sSL "https://dl.google.com/linux/direct/google-talkplugin_current_amd64.deb" -o /tmp/google-talkplugin-amd64.deb \ 
 && dpkg -i /tmp/google-talkplugin-amd64.deb \
 # Get the version of chrome (first 3 parts are required)
 && chromeVersion3Part=$(google-chrome --no-sandbox --version | grep -Po '[0-9]{1,8}.[0-9]{1,8}.[0-9]{1,8}') \
 && chromeDriverVersion=$(curl --raw "https://chromedriver.storage.googleapis.com/LATEST_RELEASE_${chromeVersion3Part}") \
 && mkdir /opt/selenium \
 && curl -sSL "https://chromedriver.storage.googleapis.com/${chromeDriverVersion}/chromedriver_linux64.zip" -o /tmp/chromedriver.zip \
 && unzip -o /tmp/chromedriver -d /opt/selenium/ \
 # clean-up
 && rm -rf /tmp/*.deb \
 && rm -rf /tmp/*.zip \
 # base image includes curl so no point in removing here unless base image changes
 && apt-get purge -y --auto-remove unzip \
 && rm -rf /var/lib/apt/lists/*

# Add chrome user
RUN groupadd -r chrome \
 && useradd -r -g chrome -G audio,video chrome \
 && mkdir -p /home/chrome/Downloads \
 && chown -R chrome:chrome /home/chrome