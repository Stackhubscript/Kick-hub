-- Server-side: ServerScriptService/KickHandler
-- Só permita executar em SEU jogo. Admins list controla quem pode usar.

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- Cria RemoteEvent se não existir
local kickEvent = ReplicatedStorage:FindFirstChild("KickEvent")
if not kickEvent then
    kickEvent = Instance.new("RemoteEvent")
    kickEvent.Name = "KickEvent"
    kickEvent.Parent = ReplicatedStorage
end

-- Lista de Admins (coloque os UserIds permitidos aqui)
local Admins = {
    [game.CreatorId] = true, -- dono do jogo por padrão
    -- Exemplo: [12345678] = true,
}

-- Função para encontrar jogador pelo nome (case-insensitive parcial/exato)
local function findPlayerByName(name)
    if not name or name == "" then return nil end
    name = name:lower()
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl.Name:lower() == name then
            return pl
        end
    end
    -- tenta partial match
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl.Name:lower():find(name, 1, true) then
            return pl
        end
    end
    return nil
end

-- Handler quando um cliente pedir para kickar
kickEvent.OnServerEvent:Connect(function(requester, targetName)
    -- Verifica permissões
    if not requester or not requester.UserId then return end
    if not Admins[requester.UserId] then
        warn(("Player %s tentou usar KickEvent sem permissão"):format(requester.Name))
        return
    end

    local target = findPlayerByName(tostring(targetName))
    if not target then
        -- opcional: informar o requester via mensagem (RemoteEvent de resposta poderia ser implementado)
        warn(("Kick requested but target not found: %s (by %s)"):format(tostring(targetName), requester.Name))
        return
    end

    -- Previne kickar outros admins (opcional)
    if Admins[target.UserId] then
        warn(("Attempt to kick an admin blocked: %s tried to kick %s"):format(requester.Name, target.Name))
        return
    end

    -- Mensagem exibida ao jogador desconectado
    local kickMessage = "You have been banned for violating the rules"

    -- Kicka o jogador do servidor
    local success, err = pcall(function()
        target:Kick(kickMessage)
    end)
    if not success then
        warn("Falha ao kickar "..tostring(target.Name)..": "..tostring(err))
    end
end)
