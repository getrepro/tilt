# Centralized Tilt setup for Repro projects (excluding SDKs).
# Run: tilt up (from this directory)

# Infrastructure
# MongoDB runs via Tilt-managed docker container (no docker-compose).
local_resource(
    'mongo',
    cmd='docker rm -f repro-mongo >/dev/null 2>&1 || true',
    serve_cmd='docker run --rm --name repro-mongo -p 27017:27017 -v repro-mongo-data:/data/db mongo:7 --quiet --logpath /dev/null --bind_ip_all',
    labels=['infra'],
)

MONGO_URI_BASE = 'mongodb://localhost:27017'
ENV_LOAD = 'set -a; [ -f .env ] && . ./.env; [ -f .env.local ] && . ./.env.local; set +a;'

def cmd_in(dir, cmd):
    return 'cd ' + dir + ' && ' + ENV_LOAD + ' ' + cmd

PORTS = {
    'repro_api': 4010,
    'repro_ai': 8000,
    'repro_web': 5174,
    'repro_demo_backend': 3008,
    'repro_demo_web': 5173,
    'repro_demo_next': 3000,
    'repro_demo_node_e2e': 3001,
    'repro_demo_nest_e2e': 3002,
    'repro_demo_react_e2e': 5175,
    'repro_demo_nest_react_e2e': 5176,
}

def localhost(port, path=''):
    if path and not path.startswith('/'):
        path = '/' + path
    return 'http://localhost:' + str(port) + path

# Core services
local_resource(
    'repro-api',
    serve_cmd=cmd_in('../api', 'PORT=' + str(PORTS['repro_api']) + ' MONGO_URI=' + MONGO_URI_BASE + '/repro MONGO_TLS=false npm run start:dev'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_api'], 'api/docs')],
    labels=['core'],
)
local_resource(
    'repro-ai',
    serve_cmd=cmd_in('../repro-ai', 'MONGO_URI=' + MONGO_URI_BASE + ' MONGO_DB=repro uvicorn app.main:app --reload --port ' + str(PORTS['repro_ai'])),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_ai'], 'docs')],
    labels=['core'],
)
local_resource(
    'repro-web',
    serve_cmd=cmd_in('../repro-web', 'VITE_API_BASE=' + localhost(PORTS['repro_api'], 'api') + ' npm run dev -- --port ' + str(PORTS['repro_web'])),
    links=[localhost(PORTS['repro_web'])],
    labels=['core'],
)

# Demo apps
local_resource(
    'repro-demo-backend',
    serve_cmd=cmd_in('../repro-demo/backend', 'PORT=' + str(PORTS['repro_demo_backend']) + ' MONGODB_URI=' + MONGO_URI_BASE + '/taskflow npm run start:dev'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_demo_backend'])],
    labels=['demo'],
)
local_resource(
    'repro-demo-web',
    serve_cmd=cmd_in('../repro-demo', 'VITE_API_URL=' + localhost(PORTS['repro_demo_backend']) + ' npm run dev -- --port ' + str(PORTS['repro_demo_web'])),
    links=[localhost(PORTS['repro_demo_web'])],
    labels=['demo'],
)
local_resource(
    'repro-demo-next',
    serve_cmd=cmd_in('../repro-next-demo', 'NEXT_PUBLIC_API_URL=' + localhost(PORTS['repro_demo_backend']) + ' npm run dev -- -p ' + str(PORTS['repro_demo_next'])),
    links=[localhost(PORTS['repro_demo_next'])],
    labels=['demo'],
)
local_resource(
    'repro-demo-node-e2e',
    serve_cmd=cmd_in('../repro-node-e2e', 'MONGO_URL=' + MONGO_URI_BASE + '/repro-e2e REPRO_API_BASE=' + localhost(PORTS['repro_api']) + ' node start.js'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_demo_node_e2e'])],
    labels=['demo'],
)
local_resource(
    'repro-demo-nest-e2e',
    serve_cmd=cmd_in('../repor-nest-e2e', 'PORT=' + str(PORTS['repro_demo_nest_e2e']) + ' MONGO_URL=' + MONGO_URI_BASE + '/repro-e2e REPRO_API_BASE=' + localhost(PORTS['repro_api']) + ' npm run dev'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_demo_nest_e2e'])],
    labels=['demo'],
)
local_resource(
    'repro-demo-react-e2e',
    serve_cmd=cmd_in('../repro-react-e2e', 'VITE_REPRO_API_BASE=' + localhost(PORTS['repro_api']) + ' VITE_NODE_BASE=' + localhost(PORTS['repro_demo_node_e2e']) + ' npm run dev -- --port ' + str(PORTS['repro_demo_react_e2e'])),
    links=[localhost(PORTS['repro_demo_react_e2e'])],
    labels=['demo'],
)
local_resource(
    'repro-demo-nest-react-e2e',
    serve_cmd=cmd_in('../repro-nest-react-e2e', 'npm run dev -- --port ' + str(PORTS['repro_demo_nest_react_e2e'])),
    links=[localhost(PORTS['repro_demo_nest_react_e2e'])],
    labels=['demo'],
)
