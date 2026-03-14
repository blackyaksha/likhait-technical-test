# syntax = docker/dockerfile:1

# Use Ruby 3.3.7
ARG RUBY_VERSION=3.3.7
FROM ruby:$RUBY_VERSION-slim

# Rails app lives here
WORKDIR /rails

# Install base packages
# Added libyaml-dev, required to build gem for Rails
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y build-essential ca-certificates default-mysql-client default-libmysqlclient-dev git libyaml-dev && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Install application gems
COPY Gemfile Gemfile.lock ./

# Configure bundler and install gems with retry logic
RUN bundle config set --local deployment 'false' && \
    bundle config set --local path 'vendor/bundle' && \
    bundle install --jobs 4 --retry 3

# Copy application code
COPY . .

# Fix potential Windows CRLF line ending issues for entrypoint scripts
RUN apt-get update -qq && apt-get install -y dos2unix && \
    dos2unix /rails/bin/docker-entrypoint && \
    chmod +x /rails/bin/docker-entrypoint

# Precompile bootsnap code for faster boot times
RUN bundle exec bootsnap precompile --gemfile app/ lib/

# Entrypoint prepares the database
# Use bash to run the entrypoint to avoid "exec format error"
# which may occur when scripts are created on Windows environments.
ENTRYPOINT ["bash", "/rails/bin/docker-entrypoint"]

# Start the server by default
# Added "bundle" and "exec" to ensure the server runs with the correct gem environment
# gems are installed to "vendor/bundle" which is not in the default path, resulting to adding "bundle" and "exec"
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]