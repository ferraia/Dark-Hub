
-- SISTEMA DE TESTE ROBLOX
-- Para seu próprio jogo no Roblox Studio

local Players = game:GetService("Players")
local ovosFolder = workspace:WaitForChild("Eggs")

local velocidadeMaxima = 32
local duracaoEscudo = 8
local eventoAtivo = false
local escudos = {}

-- Criar placar
Players.PlayerAdded:Connect(function(player)
    local stats = Instance.new("Folder")
    stats.Name = "leaderstats"
    stats.Parent = player

    local ovos = Instance.new("IntValue")
    ovos.Name = "Ovos"
    ovos.Value = 0
    ovos.Parent = stats

    player.CharacterAdded:Connect(function(character)
        local humanoid = character:WaitForChild("Humanoid")
        humanoid.WalkSpeed = 16
    end)
end)

-- Coleta instantânea
local function configurarOvo(ovo)
    if not ovo:IsA("BasePart") then return end

    local prompt = Instance.new("ProximityPrompt")
    prompt.ActionText = "Coletar"
    prompt.ObjectText = "Ovo"
    prompt.HoldDuration = 0
    prompt.Parent = ovo

    local coletado = false

    prompt.Triggered:Connect(function(player)
        if coletado then return end

        local stats = player:FindFirstChild("leaderstats")
        local contador = stats and stats:FindFirstChild("Ovos")
        if not contador then return end

        coletado = true
        contador.Value += eventoAtivo and 2 or 1
        ovo:Destroy()
    end)
end

for _, ovo in ipairs(ovosFolder:GetChildren()) do
    configurarOvo(ovo)
end

ovosFolder.ChildAdded:Connect(configurarOvo)

-- Velocidade limitada
local function definirVelocidade(player, valor)
    local character = player.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")

    if humanoid then
        humanoid.WalkSpeed = math.clamp(valor, 0, velocidadeMaxima)
    end
end

-- Escudo temporário
local function ativarEscudo(player)
    local expiracao = os.clock() + duracaoEscudo
    escudos[player] = expiracao

    task.delay(duracaoEscudo, function()
        if escudos[player] == expiracao then
            escudos[player] = nil
        end
    end)
end

local function estaProtegido(player)
    return escudos[player] ~= nil
        and escudos[player] > os.clock()
end

-- Evento de ovos em dobro por 30 segundos
local function iniciarEvento()
    if eventoAtivo then return end

    eventoAtivo = true
    print("Evento iniciado: ovos em dobro!")

    task.wait(30)

    eventoAtivo = false
    print("Evento encerrado!")
end

Players.PlayerRemoving:Connect(function(player)
    escudos[player] = nil
end)

-- Testes no servidor:
-- definirVelocidade(player, 32)
-- ativarEscudo(player)
-- print(estaProtegido(player))
-- iniciarEvento()
