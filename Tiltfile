# Centralized Tilt setup for Repro projects (excluding SDKs).
# Run: tilt up (from this directory)

# Infrastructure
# MongoDB runs via Tilt-managed docker container (no docker-compose).
local_resource(
    'mongo',
    cmd='docker rm -f repro-mongo >/dev/null 2>&1 || true',
    serve_cmd='docker run --rm --name repro-mongo -p 27017:27017 -v repro-mongo-data:/data/db mongo:7 --quiet --logpath /dev/null --bind_ip_all',
)

MONGO_URI_BASE = 'mongodb://localhost:27017'
ENV_LOAD = 'set -a; [ -f .env ] && . ./.env; [ -f .env.local ] && . ./.env.local; set +a;'

def cmd_in(dir, cmd):
    return 'cd ' + dir + ' && ' + ENV_LOAD + ' ' + cmd

PORTS = {
    'api': 4010,
    'repro_ai': 8000,
    'repro_demo_backend': 3008,
    'repro_node_e2e': 3001,
    'repor_nest_e2e': 3002,
    'repro_demo_web': 5173,
    'repro_web': 5174,
    'repro_next_demo': 3000,
    'repro_react_e2e': 5175,
    'repro_nest_react_e2e': 5176,
}

def localhost(port, path=''):
    if path and not path.startswith('/'):
        path = '/' + path
    return 'http://localhost:' + str(port) + path

# Backend services
local_resource(
    'api',
    serve_cmd=cmd_in('../api', 'PORT=' + str(PORTS['api']) + ' MONGO_URI=' + MONGO_URI_BASE + '/repro MONGO_TLS=false npm run start:dev'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['api'], 'api/docs')],
)
local_resource(
    'repro-ai',
    serve_cmd=cmd_in('../repro-ai', 'MONGO_URI=' + MONGO_URI_BASE + ' MONGO_DB=repro uvicorn app.main:app --reload --port ' + str(PORTS['repro_ai'])),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_ai'], 'docs')],
)
local_resource(
    'repro-demo-backend',
    serve_cmd=cmd_in('../repro-demo/backend', 'PORT=' + str(PORTS['repro_demo_backend']) + ' MONGODB_URI=' + MONGO_URI_BASE + '/taskflow npm run start:dev'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_demo_backend'])],
)
local_resource(
    'repor-nest-e2e',
    serve_cmd=cmd_in('../repor-nest-e2e', 'PORT=' + str(PORTS['repor_nest_e2e']) + ' MONGO_URL=' + MONGO_URI_BASE + '/repro-e2e REPRO_API_BASE=' + localhost(PORTS['api']) + ' npm run dev'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repor_nest_e2e'])],
)
local_resource(
    'repro-node-e2e',
    serve_cmd=cmd_in('../repro-node-e2e', 'MONGO_URL=' + MONGO_URI_BASE + '/repro-e2e REPRO_API_BASE=' + localhost(PORTS['api']) + ' node start.js'),
    resource_deps=['mongo'],
    links=[localhost(PORTS['repro_node_e2e'])],
)

# Frontend apps
local_resource(
    'repro-demo-web',
    serve_cmd=cmd_in('../repro-demo', 'VITE_API_URL=' + localhost(PORTS['repro_demo_backend']) + ' npm run dev -- --port ' + str(PORTS['repro_demo_web'])),
    links=[localhost(PORTS['repro_demo_web'])],
)
local_resource(
    'repro-web',
    serve_cmd=cmd_in('../repro-web', 'VITE_API_BASE=' + localhost(PORTS['api'], 'api') + ' npm run dev -- --port ' + str(PORTS['repro_web'])),
    links=[localhost(PORTS['repro_web'])],
)
local_resource(
    'repro-next-demo',
    serve_cmd=cmd_in('../repro-next-demo', 'NEXT_PUBLIC_API_URL=' + localhost(PORTS['repro_demo_backend']) + ' npm run dev -- -p ' + str(PORTS['repro_next_demo'])),
    links=[localhost(PORTS['repro_next_demo'])],
)
local_resource(
    'repro-react-e2e',
    serve_cmd=cmd_in('../repro-react-e2e', 'VITE_REPRO_API_BASE=' + localhost(PORTS['api']) + ' VITE_NODE_BASE=' + localhost(PORTS['repro_node_e2e']) + ' npm run dev -- --port ' + str(PORTS['repro_react_e2e'])),
    links=[localhost(PORTS['repro_react_e2e'])],
)
local_resource(
    'repro-nest-react-e2e',
    serve_cmd=cmd_in('../repro-nest-react-e2e', 'npm run dev -- --port ' + str(PORTS['repro_nest_react_e2e'])),
    links=[localhost(PORTS['repro_nest_react_e2e'])],
)
