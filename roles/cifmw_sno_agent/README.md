# cifmw_sno_agent

Prepare **Single Node OpenShift (SNO)** using the **OpenShift agent-based installer**:
render `install-config.yaml` and `agent-config.yaml`, run `openshift-install agent create
cluster-manifests` and `openshift-install agent create image`, and optionally wait for
`install-complete`.

This is appropriate when the OpenShift node is a **single VM or bare-metal machine**
(no nested virtualization required). You attach the generated **agent ISO** to that
machine and boot it; the node is **re-imaged** as RHCOS/OpenShift (any prior OS on the
disk is replaced when you install to the hinted disk).

## Requirements

- `openshift-install` on the **target host** (the play’s inventory host), or
  outbound HTTPS so the role can download it from `mirror.openshift.com` when
  `cifmw_sno_openshift_install_download` is true (default).
- `pull-secret.json` on the **Ansible controller**; path passed as
  `cifmw_sno_pull_secret_src`.
- For static IP, fill `network_config` under the matching `cifmw_sno_hosts` entry
  (NMState shape as in OpenShift agent docs).

## Main variables

| Variable | Meaning |
| -------- | ------- |
| `cifmw_sno_pull_secret_src` | Path to pull secret on the controller (required). |
| `cifmw_sno_cluster_name` | Cluster name (`metadata.name`). |
| `cifmw_sno_base_domain` | DNS base domain. |
| `cifmw_sno_rendezvous_ip` | Rendezvous IP (for SNO, the node’s IP). |
| `cifmw_sno_hosts` | List of nodes: `hostname`, `interface_name`, `mac_address`, optional `network_config`, `root_device_hint_device`. |
| `cifmw_sno_machine_network_cidr` | Machine network CIDR (must include rendezvous IP). |
| `cifmw_sno_workdir` | Directory for assets (default under `ci-framework-data/sno-agent/<cluster_name>`). |
| `cifmw_sno_openshift_install_download` | Download `openshift-install` into `cifmw_sno_openshift_install_tools_dir` when missing (default `true`). |
| `cifmw_sno_openshift_client_version` | Mirror path segment, e.g. `4.16.12` or `stable-4.16` (required when download is enabled and the binary is absent). |
| `cifmw_sno_openshift_install_tarball_url` | Optional full URL to the `openshift-install` tarball (skips URL composition). |
| `cifmw_sno_openshift_install_tools_dir` | Where the downloaded binary is stored and reused. |
| `cifmw_sno_build_iso` | Run `agent create cluster-manifests` and `agent create image` (default `true`). |
| `cifmw_sno_wait_install_complete` | After booting from ISO, run `wait-for` on the install host (default `false`). |
| `cifmw_sno_release_image_override` | Optional `OPENSHIFT_INSTALL_RELEASE_IMAGE_OVERRIDE` value. |

## Tags

- `cifmw_sno_prepare` — workdir, pull secret copy, templates
- `cifmw_sno_build` — openshift-install agent image build
- `cifmw_sno_wait` — wait-for bootstrap/install

## Playbook

See `playbooks/deploy-sno-agent.yml` and `scenarios/reproducers/sno-agent-example.yml`.

## ShiftStack

OpenStack **IPI** (often associated with “ShiftStack”) targets a **Nova/Neutron** API.
This role does **not** replace that flow: it is for **agent-based** installs on a
machine you boot from ISO (VM or bare metal).
