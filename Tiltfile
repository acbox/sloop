load('ext://helm_resource', 'helm_resource')

cluster_name = 'kind-sloop'
sloop = 'sloop'

# Allow the cluster to avoid problems while having kubectl configured to talk to a remote cluster.
allow_k8s_contexts(cluster_name)

# Load the restart_process extension with the docker_build_with_restart func for live reloading.
load('ext://restart_process', 'docker_build_with_restart')

# Building binary locally.
local_resource('sloop-binary',
    'GOOS=linux go build -o sloop ./pkg/sloop',
    deps=[
        './pkg/sloop/',
    ],
)

# Use custom Dockerfile for Tilt builds, which only takes locally built binary for live reloading.
dockerfile = '''
    FROM golang:1.19
    RUN go install github.com/go-delve/delve/cmd/dlv@latest
    COPY sloop /sloop
    '''

# Wrap a docker_build to restart the given entrypoint after a Live Update.
docker_build_with_restart(
    sloop,
    '.',
    dockerfile_contents=dockerfile,
    entrypoint='/sloop',
    #entrypoint='/go/bin/dlv --listen=0.0.0.0:50100 --api-version=2 --headless=true --only-same-user=false --accept-multiclient --check-go-version=false exec /sloop --display-context cluster',
    live_update=[
        # Copy the binary so it gets restarted.
        sync(sloop, '/sloop'),
    ],
)

# Deploy resources via Helm chart but using in-development Sloop image
helm_resource(
  'sloop-helm',
  './helm/sloop',
  deps=['./sloop'],
  image_keys=[('image.repository', 'image.tag')],
  image_deps=[sloop],
  port_forwards=["50100:50100","8080:8080"],
  #flags=['--set','displayContext=cluster'], # doesn't work
)
