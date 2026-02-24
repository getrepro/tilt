# Full local dev setup for all repos in ../repro.
# This Tiltfile always runs the full stack (infra + SDK watch + apps).

MONGO_URI_BASE = 'mongodb://localhost:27017'
ENV_LOAD = 'set -a; [ -f .env ] && . ./.env; [ -f .env.local ] && . ./.env.local; set +a;'

PORTS = {
    'api': 4010,
    'repro_web': 5174,
    'repro_demo_backend': 3008,
    'repro_demo_web': 5173,
    'log_collector': 8080,
    'kafka': 9092,
    'kafka_ui': 8081,
    'clickhouse_http': 8123,
    'clickhouse_native': 9000,
}

def cmd_in(dir, cmd):
    return 'cd ' + dir + ' && ' + ENV_LOAD + ' ' + cmd

def localhost(port, path=''):
    if path and not path.startswith('/'):
        path = '/' + path
    return 'http://localhost:' + str(port) + path

# Shared infrastructure (mongo + kafka + kafka-ui + clickhouse)
local_resource(
    'infra',
    serve_cmd='docker compose -f ./docker-compose.infra.yml up --remove-orphans',
    labels=['infra'],
    links=[
        localhost(PORTS['kafka_ui']),
        localhost(PORTS['clickhouse_http']),
    ],
)

# SDK watch pipelines
local_resource(
    'node-sdk',
    cmd=cmd_in('../node-sdk', 'npm run build'),
    serve_cmd=cmd_in('../node-sdk', 'npm run dev'),
    labels=['sdk'],
)

local_resource(
    'react-sdk',
    cmd=cmd_in('../react-sdk', 'npm run build'),
    serve_cmd=cmd_in('../react-sdk', 'npm run dev'),
    labels=['sdk'],
)

# Ingest pipeline
local_resource(
    'log-collector',
    serve_cmd=cmd_in(
        '../log-collector',
        'HOST=0.0.0.0 PORT=' + str(PORTS['log_collector']) + ' SERVICE_MODE=all KAFKA_BROKERS=127.0.0.1:' + str(PORTS['kafka']) + ' CLICKHOUSE_URL=' + localhost(PORTS['clickhouse_http']) + ' CLICKHOUSE_DATABASE=repro CLICKHOUSE_USERNAME=repro CLICKHOUSE_PASSWORD=repro npm run watch',
    ),
    resource_deps=['infra'],
    links=[localhost(PORTS['log_collector'], 'health/live')],
    labels=['services'],
)

# Core API + console
local_resource(
    'repro-api',
    serve_cmd=cmd_in(
        '../api',
        'PORT=' + str(PORTS['api']) + ' MONGO_URI=' + MONGO_URI_BASE + '/repro MONGO_TLS=false DATA_ENCRYPTION_KEY=${DATA_ENCRYPTION_KEY:-repro-local-dev-key} npm run start:dev',
    ),
    resource_deps=['infra'],
    links=[localhost(PORTS['api'], 'api/docs')],
    labels=['apps'],
)

local_resource(
    'repro-web',
    serve_cmd=cmd_in(
        '../repro-web',
        'VITE_API_BASE=' + localhost(PORTS['api'], 'api') + ' npm run dev -- --port ' + str(PORTS['repro_web']),
    ),
    resource_deps=['repro-api'],
    links=[localhost(PORTS['repro_web'])],
    labels=['apps'],
)

# Demo stack wired to local SDKs
local_resource(
    'repro-demo-backend',
    serve_cmd=cmd_in(
        '../repro-demo/backend',
        'PORT=' + str(PORTS['repro_demo_backend']) + ' MONGODB_URI=' + MONGO_URI_BASE + '/taskflow REPRO_INGEST_BASE=' + localhost(PORTS['log_collector']) + ' npm run start:dev',
    ),
    deps=['../node-sdk/dist'],
    resource_deps=['infra', 'node-sdk', 'log-collector'],
    links=[localhost(PORTS['repro_demo_backend'])],
    labels=['demo'],
)

local_resource(
    'repro-demo-web',
    serve_cmd=cmd_in(
        '../repro-demo',
        'VITE_API_URL=' + localhost(PORTS['repro_demo_backend']) + ' npm run dev -- --port ' + str(PORTS['repro_demo_web']),
    ),
    deps=['../react-sdk/dist'],
    resource_deps=['repro-demo-backend', 'react-sdk'],
    links=[localhost(PORTS['repro_demo_web'])],
    labels=['demo'],
)
