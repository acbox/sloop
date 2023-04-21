load('ext://helm_resource', 'helm_resource')

# Allow the cluster to avoid problems while having kubectl configured to talk to a remote cluster.
allow_k8s_contexts('kind-sloop')

# Load the restart_process extension with the docker_build_with_restart func for live reloading.
load('ext://restart_process', 'docker_build_with_restart')

# Building binary locally.
local_resource('sloop-binary',
    # gcflags disable all optimisations
    'GOOS=linux go build -gcflags "all=-N -l" -o sloop pkg/sloop/main.go',
    deps=[
        './pkg/sloop/',
    ],
)

# Use custom Dockerfile for Tilt builds, which only takes locally built binary for live reloading.
dockerfile = '''
    FROM golang:1.19-alpine
    RUN go install github.com/go-delve/delve/cmd/dlv@latest
    COPY sloop /sloop
    '''

# Wrap a docker_build to restart the given entrypoint after a Live Update.
docker_build_with_restart(
    'sloop-image',
    '.',
    dockerfile_contents=dockerfile,
    entrypoint='/sloop',
    #entrypoint='/go/bin/dlv --listen=0.0.0.0:50100 --api-version=2 --headless=true --only-same-user=false --accept-multiclient --check-go-version=false exec /sloop',
    live_update=[
        # Copy the binary so it gets restarted.
        sync('sloop', '/sloop'),
    ],
)

# Deploy resources via Helm chart, replacing image with local build under development
helm_resource(
  'sloop-helm',
  './helm/sloop',
  deps=['./sloop'],
  image_keys=[('image.repository', 'image.tag')],
  image_deps=['sloop-image'],
  port_forwards=["50100:50100","8080:8080"],
)
