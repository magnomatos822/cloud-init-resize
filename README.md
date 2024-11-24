# Cloud-Init Disk Expansion Configuration para Ubuntu 24.04

Esta configuração permite o redimensionamento automático do disco em VMs Ubuntu 24.04 usando cloud-init no ambiente VMware vSphere.

## Pré-requisitos

- Ubuntu 24.04 Server ISO
- VMware vSphere
- Terraform
- Cloud-init instalado na imagem base

## Como usar

1. **Preparação da Imagem Base**:
   - Instale o Ubuntu 24.04 Server em uma VM
   - Certifique-se que o cloud-init está instalado:
     ```bash
     sudo apt update
     sudo apt install -y cloud-init
     ```

2. **Configuração do Cloud-Init**:
   - Copie o arquivo `cloud-init-config.yaml` para `/etc/cloud/cloud.cfg.d/99_resize.cfg` na VM base
   - Execute:
     ```bash
     sudo cloud-init clean
     ```

3. **Configuração no Terraform**:
   ```hcl
   resource "vsphere_virtual_machine" "vm" {
     # ... outras configurações ...

     disk {
       label = "disk0"
       size  = var.disk_size  # Tamanho desejado em GB
     }

     extra_config = {
       "guestinfo.metadata"          = base64encode(file("cloud-init-config.yaml"))
       "guestinfo.metadata.encoding" = "base64"
     }
   }
   ```

4. **Verificação**:
   - Após a criação da VM, verifique o status do cloud-init:
     ```bash
     cloud-init status
     ```
   - Verifique o tamanho do disco:
     ```bash
     df -h /
     ```

## Notas Importantes

- O cloud-init irá automaticamente redimensionar o disco na primeira inicialização
- O processo é executado apenas uma vez por instância
- Certifique-se que o sistema de arquivos suporta redimensionamento (ext4, xfs)

## Troubleshooting

Se o redimensionamento não ocorrer automaticamente:

1. Verifique os logs do cloud-init:
   ```bash
   sudo cat /var/log/cloud-init.log
   ```

2. Execute manualmente o redimensionamento:
   ```bash
   sudo growpart /dev/sda 1
   sudo resize2fs /dev/sda1  # para ext4
   # ou
   sudo xfs_growfs /  # para XFS
   ```
