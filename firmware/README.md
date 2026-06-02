# Firmware ZMK

Este repositório contém agora a configuração-fonte ZMK do PCBLander.

Os `.uf2` antigos nesta pasta são firmwares já compilados do `Keyboardv3`. A configuração nova está nestes ficheiros:

- `build.yaml`
- `config/pcblander.conf`
- `config/pcblander.keymap`
- `boards/shields/pcblander/`
- `.github/workflows/build.yml`
- `zephyr/module.yml`

## Layout

O keymap atual tem 4 camadas, inspirado no layout Moonlander que enviaste:

- `Base`: letras e símbolos principais.
- `Nav`: navegação, setas, `Home`, `End`, `Page Up`, `Page Down`, `F1` a `F12` e Bluetooth.
- `Num`: símbolos e numpad no lado direito.
- `Play`: camada simples para jogo, com letras diretas e menos teclas especiais.

Os botões de polegar principais usam:

- `Desktop`: `Gui+D`.
- `Alfred`: `Alt+Space`.
- `TO 1`: muda para `Nav`.
- `TO 2`: muda para `Num`.
- `TO 3`: muda para `Play`.

## Compilar pelo GitHub Actions

Este é o método mais simples.

1. Faz commit e push destes ficheiros para o GitHub.
2. Abre o repositório no GitHub.
3. Vai a `Actions`.
4. Abre o workflow `Build ZMK firmware`.
5. Quando acabar, descarrega o artifact chamado `firmware`.
6. Extrai o `.zip`.

Devem aparecer:

- `pcblander_left.uf2`
- `pcblander_right.uf2`

O ficheiro `pcblander_left.uf2` já é compilado com suporte para ZMK Studio por USB. O lado direito não precisa de Studio, porque num split wireless o lado esquerdo é o central.

## Compilar localmente com Docker

Instala o Docker Desktop e depois abre PowerShell na raiz do projeto.

```powershell
docker run --rm -it -v "${PWD}:/workspace" -w /workspace zmkfirmware/zmk-build-arm:stable bash
```

Dentro do container:

```bash
west init -l config
west update --fetch-opt=--filter=tree:0
west zephyr-export

west build -s zmk/app -d build/left -b nice_nano//zmk \
  -S studio-rpc-usb-uart -- \
  -DZMK_CONFIG=/workspace/config \
  -DZMK_EXTRA_MODULES=/workspace \
  -DCONFIG_ZMK_STUDIO=y \
  -DSHIELD=pcblander_left

west build -s zmk/app -d build/right -b nice_nano//zmk -- \
  -DZMK_CONFIG=/workspace/config \
  -DZMK_EXTRA_MODULES=/workspace \
  -DSHIELD=pcblander_right
```

Depois de compilar, os ficheiros ficam em:

- `build/left/zephyr/zmk.uf2`
- `build/right/zephyr/zmk.uf2`

## Flash

1. Liga a metade esquerda ao USB.
2. Entra em bootloader no nice!nano.
3. Copia `pcblander_left.uf2` para a drive que aparecer.
4. Repete na metade direita com `pcblander_right.uf2`.
5. Liga as duas metades e faz reset nas duas quase ao mesmo tempo para emparelhar o split.

Assunções atuais:

- `pcblander_left` é a metade central.
- `pcblander_right` é a metade periférica.
- Ambas usam nice!nano/nRF52840.
- A matriz usa diodos `col2row`.
