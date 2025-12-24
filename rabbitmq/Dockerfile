FROM rabbitmq:3-management

# Fix for Windows Docker Desktop erlang cookie permission issue
# Pre-create the cookie file with correct permissions before RabbitMQ starts
RUN rabbitmq-plugins enable --offline rabbitmq_management && \
    echo "hotelier-secret-cookie" > /var/lib/rabbitmq/.erlang.cookie && \
    chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie && \
    chmod 400 /var/lib/rabbitmq/.erlang.cookie
