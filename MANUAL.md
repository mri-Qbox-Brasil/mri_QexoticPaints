# mri_QexoticPaints — Manual

Latas de spray usáveis que aplicam as cores exóticas (anodizadas, flip, pearl, prismáticas, holográficas) do game build 2699+ em veículos, com animação de pintura e persistência no banco.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Itens](#itens)
4. [Como funciona](#como-funciona)
5. [Cores disponíveis](#cores-disponíveis)
6. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
7. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `qb-core` | Sim | O código chama `exports['qb-core']:GetCoreObject()` no cliente e no servidor |
| `oxmysql` | Sim | Carregado no `fxmanifest.lua` (`@oxmysql/lib/MySQL.lua`); grava a pintura em `player_vehicles` |

O recurso também exige **game build 2699 ou superior**. O `client.lua` verifica `GetGameBuildNumber()` na inicialização e aborta com a mensagem `YOU NEED AT LEAST 2699 GAME BUILD` se o build for menor.

---

## Instalação

1. Copie a pasta `mri_QexoticPaints` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure mri_QexoticPaints
   ```
3. Garanta que o servidor roda com build **2699+** (`sv_enforceGameBuild 2699` ou superior).
4. Cadastre os itens de spray no inventário — um item por cor, com o **nome exatamente igual à chave** da tabela `Paints` em `colors.lua`. Veja [Itens](#itens).
5. O recurso deve iniciar **antes** do jogador entrar no servidor: os arquivos `meta/carcols_gen9.meta` e `meta/carmodcols_gen9.meta` são registrados como `data_file` e precisam estar carregados para as cores existirem.

---

## Itens

Cada cor é um item usável separado. O servidor registra todos automaticamente:

```lua
for i, d in pairs(Paints) do
    QBCore.Functions.CreateUseableItem(i, function(source, item)
        TriggerClientEvent("lambra-exoticPaints:client:usedSpray", source, i)
    end)
end
```

Ou seja, **o nome do item precisa ser idêntico à chave da tabela `Paints`** (`anod_red`, `ykta_hsw`, etc.). Se o item não existir no inventário, a lata simplesmente não pode ser usada.

Exemplo de item (formato `qb-core/shared/items.lua`):

```lua
['anod_red'] = { ['name'] = 'anod_red', ['label'] = 'Exotic Spray can', ['weight'] = 200, ['type'] = 'item', ['image'] = 'exoticspray.png', ['unique'] = false, ['useable'] = true, ['description'] = 'Anod Red' },
```

O item deve ser `useable = true`. A lista completa de nomes está em [Cores disponíveis](#cores-disponíveis) e o `README.md` do repositório traz o bloco pronto com as 81 entradas.

---

## Como funciona

1. O jogador usa a lata de spray **fora do veículo**, a até 5 metros do carro.
2. O script pega o veículo mais próximo (`GetClosestVehicle`, raio 5.0), congela ele, e toca a animação de pintura com o prop `prop_cs_spray_can` na mão.
3. A carga da lata começa em **100%** e é exibida em texto 3D acima do prop.
4. Segurando o **botão de ataque** (mouse esquerdo), o jogador pinta: a carga cai **3% por segundo** e sai o efeito de partícula `scr_lamgraff_paint_spray`.
5. Ao soltar o botão, a lata precisa ser **chacoalhada** antes de pintar de novo — segure o **botão de mira** (mouse direito) para reproduzir a animação de agitar a lata.
6. Quando a carga chega a **0%**, a pintura é aplicada: `SetVehicleColours(veh, Paints[item], Paints[item])` define a cor primária e secundária.
7. O item é consumido no servidor (`RemoveItem`) e, se a placa existir em `player_vehicles`, as propriedades do veículo são gravadas na coluna `mods`.

Sair do raio de 5 metros do veículo cancela o processo: o prop é apagado, o veículo é descongelado e nada é aplicado nem consumido.

---

## Cores disponíveis

A tabela `Paints` (em `colors.lua`) mapeia o **nome do item** para o **índice de cor** do GTA. É esse arquivo que você edita para adicionar ou remover cores.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Paints['<nome_do_item>']` | number | Sim | Índice de cor do veículo aplicado por `SetVehicleColours`. A chave é o nome do item no inventário |

Cores registradas (item → índice):

| Item | Índice | Item | Índice |
|---|---|---|---|
| `anod_red` | 161 | `red_orangeflip` | 190 |
| `anod_wine` | 162 | `orang_purp_flip` | 191 |
| `anod_purple` | 163 | `orang_blue_flip` | 192 |
| `anod_blue` | 164 | `white_purp_flip` | 193 |
| `anod_green` | 165 | `red_rainbo_flip` | 194 |
| `anod_lime` | 166 | `blu_rainbo_flip` | 195 |
| `anod_copper` | 167 | `darkgreenpearl` | 196 |
| `anod_bronze` | 168 | `darktealpearl` | 197 |
| `anod_champagne` | 169 | `darkbluepearl` | 198 |
| `anod_gold` | 170 | `darkpurplepearl` | 199 |
| `green_blue_flip` | 171 | `oil_slick_pearl` | 200 |
| `green_red_flip` | 172 | `lit_green_pearl` | 201 |
| `green_brow_flip` | 173 | `lit_blue_pearl` | 202 |
| `green_turq_flip` | 174 | `lit_purp_pearl` | 203 |
| `green_purp_flip` | 175 | `lit_pink_pearl` | 204 |
| `teal_purp_flip` | 176 | `pink_pearl` | 206 |
| `turq_red_flip` | 177 | `yellow_pearl` | 207 |
| `turq_purp_flip` | 178 | `green_pearl` | 208 |
| `cyan_purp_flip` | 179 | `blue_pearl` | 209 |
| `blue_pink_flip` | 180 | `cream_pearl` | 210 |
| `blue_green_flip` | 181 | `white_prisma` | 211 |
| `purp_red_flip` | 182 | `graphite_prisma` | 212 |
| `purp_green_flip` | 183 | `darkblueprisma` | 213 |
| `magen_gree_flip` | 184 | `darkpurpprisma` | 214 |
| `magen_yell_flip` | 185 | `hot_pink_prisma` | 215 |
| `burg_green_flip` | 186 | `red_prisma` | 216 |
| `magen_cyan_flip` | 187 | `green_prisma` | 217 |
| `coppe_purp_flip` | 188 | `black_prisma` | 218 |
| `magen_oran_flip` | 189 | `oil_slic_prisma` | 219 |

| Item | Índice | Item | Índice |
|---|---|---|---|
| `rainbow_prisma` | 220 | `ykta_bubblegum` | 231 |
| `black_holo` | 221 | `ykta_full_rbow` | 232 |
| `white_holo` | 222 | `ykta_sunsets` | 233 |
| `ykta_monochrome` | 223 | `ykta_the_seven` | 234 |
| `ykta_nite_day` | 224 | `ykta_kamenrider` | 235 |
| `ykta_verlierer2` | 225 | `ykta_chromabera` | 236 |
| `ykta_sprunk_ex` | 226 | `ykta_christmas` | 237 |
| `ykta_vice_city` | 227 | `ykta_temperatur` | 238 |
| `ykta_synthwave` | 228 | `ykta_hsw` | 239 |
| `ykta_four_seaso` | 229 | `ykta_electro` | 240 |
| `ykta_m9_throwba` | 230 | `ykta_monika` | 241 |
| | | `ykta_fubuki` | 242 |

O índice `205` não é usado.

---

## Entrypoints para outros recursos

### Evento `lambra-exoticPaints:client:usedSpray` (cliente)

Inicia o modo de pintura no cliente. É o evento disparado pelo item usável, mas pode ser chamado por qualquer recurso do servidor.

```lua
TriggerClientEvent('lambra-exoticPaints:client:usedSpray', source, 'anod_red')
```

O parâmetro é a chave da tabela `Paints`. Se o jogador já estiver em modo de pintura, ou não houver veículo a menos de 5 metros, o evento é ignorado.

### Evento `lambra-exoticPaints:client:finishedSpray` (servidor)

Apesar do nome, é um **evento de servidor** (`RegisterNetEvent` no `server.lua`). O cliente o dispara ao concluir a pintura; ele consome o item e persiste as `mods` do veículo em `player_vehicles` quando a placa é de um veículo registrado.

```lua
TriggerServerEvent('lambra-exoticPaints:client:finishedSpray', vehicleProps, 'anod_red')
```

---

## Estrutura de arquivos

```
mri_QexoticPaints/
├── client.lua                    — modo de pintura: prop, animações, partículas, carga da lata, SetVehicleColours
├── server.lua                    — registra os itens usáveis, consome o item e grava as mods em player_vehicles
├── colors.lua                    — tabela Paints: nome do item → índice de cor (shared)
├── meta/
│   ├── carcols_gen9.meta         — definição das cores exóticas (data_file CARCOLS_GEN9_FILE)
│   └── carmodcols_gen9.meta      — definição das cores de mods (data_file CARMODCOLS_GEN9_FILE)
├── stream/
│   └── vehicle_paint_ramps.ytd   — texturas das rampas de cor
└── fxmanifest.lua
```
