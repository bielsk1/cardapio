---

# Script de Cardápio Interativo para GTA V (FiveM)

Este script implementa um sistema de cardápio interativo para servidores de GTA V utilizando a plataforma FiveM. O jogador pode abrir um cardápio.

## Funcionalidades

- Exibe um jornal na mão do jogador ao abrir o cardápio.
- O jogador pode interagir com o cardápio pressionando a tecla **E** quando estiver próximo de um ponto específico no mapa.
- Animação personalizada e controle de objetos para melhorar a imersão do jogador.

## Requisitos

- [FiveM](https://fivem.net/) - Framework para servidores de GTA V.

## Instalação

1. **Clone o repositório ou baixe o script:**
   - Coloque o script na pasta `resources` do seu servidor FiveM.

2. **Configuração do `fxmanifest.lua`:**
   - Certifique-se de adicionar este script ao seu arquivo `fxmanifest.lua` ou `__resource.lua` na pasta do recurso:
     ```lua
     client_scripts {
         'nome_do_script.lua'
     }
     ```

3. **Adicione as localizações:**
   - Configure as localizações onde o jogador pode abrir o cardápio, ajustando o arquivo `Config.Locations` dentro do script.

4. **Inicie o recurso no servidor:**
   - No arquivo de configuração do servidor (`server.cfg`), adicione:
     ```lua
     start nome_do_recurso
     ```

## Uso

1. **Abrindo o cardápio:**
   - Quando o jogador se aproxima de uma das localizações configuradas, uma mensagem aparecerá na tela instruindo-o a pressionar **E** para abrir o cardápio.
   - O jogador verá uma animação do personagem segurando um jornal, e o cardápio será exibido na tela.

2. **Fechando o cardápio:**
   - Para fechar o cardápio, o jogador pode clicar no botão de fechamento na interface do NUI. O jornal será removido da mão do personagem e a animação será interrompida.

## Código de Exemplo

```lua
local cRP = {}

cRP.Open = function()
    SendNUIMessage({
        action = "open"
    })
	newspaper = CreateObject(GetHashKey("prop_cliff_paper"), 0, 0, 0, true, true, true)
	RequestAnimDict("missfam4")
	while not HasAnimDictLoaded("missfam4") do Citizen.Wait(5) end
	TaskPlayAnim(PlayerPedId(), "missfam4", "base", 3.0, 2.0, -1, 33, 0.0, false, false, false)
	AttachEntityToEntity(newspaper, PlayerPedId(), GetPedBoneIndex(GetPlayerPed(-1), 18905), 0.26, 0.06, 0.16, 320.0, 310.0, 0.0, true, true, false, true, 1, true)
    SetNuiFocus(true, true)
    display = true
end

RegisterNUICallback('close', function()
    SetNuiFocus(false, false)
	ClearPedTasks(PlayerPedId())
	Citizen.Wait(200)
	DeleteObject(newspaper)
    display = false
end)

RegisterNetEvent('cardapio:open',function()
	cRP.Open()
end)

Citizen.CreateThread(function ()
	while true do
		Citizen.Wait(5)
		local playerPed = PlayerPedId()
		local pos = GetEntityCoords(playerPed)
		for k, v in pairs(Config.Locations) do
			local dist = #(pos - vector3(Config.Locations[k].Locations.x, Config.Locations[k].Locations.y, Config.Locations[k].Locations.z))
			if dist < 1.5 then
				DrawText3D(Config.Locations[k].Location.x, Config.Locations[k].Location.y, Config.Locations[k].Location.z + 0.5, "Aperte ~g~E~w~ ~n~ para abrir o Cardapio.")
				if IsControlJustReleased(0, 38) then
					cRP.Open()
				end
			end
		end
	end
end)

function DrawText3D(x, y, z, text) 
	SetTextScale(0.35, 0.35)
	SetTextFont(4)
	SetTextProportional(1)
	SetTextColour(255, 255, 255, 215)
	SetTextEntry("STRING")
	SetTextCentre(true)
	AddTextComponentString(text)
	SetDrawOrigin(x, y, z, 0)
	DrawText(0.0, 0.0)
	local textl = string.len(text)
	local t = {}
	t[1] = 0
    local i = 0
	local count = 1
    while true do
		i = string.find(text, " %~%n%~", i+1) 
		if i == nil then break end
		t[count + 1] = i	  
		count = count + 1
    end
	t[count + 1] = textl
	width = t[1]
	for lin = 2, #t do
		if(t[lin] - t[lin -1]) > width then
			width = t[lin] - t[lin -1]
		end
	end
	local factor = width / 370
	DrawRect(0.0, 0.0 + (0.01 * count) + 0.0025, 0.017 + factor, 0.02 * count + 0.01, 0, 0, 0, 127)
	ClearDrawOrigin()
end
```

---
