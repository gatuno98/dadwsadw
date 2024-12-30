-- Função para criar uma linha entre dois jogadores
local function criarLinhaEntreJogadores(player1, player2)
    -- Cria uma linha (Part) entre os jogadores
    local linha = Instance.new("Part")
    linha.Size = Vector3.new(0.2, 0.2, (player1.Character.HumanoidRootPart.Position - player2.Character.HumanoidRootPart.Position).Magnitude)
    linha.Anchored = true
    linha.CanCollide = false
    linha.Color = Color3.fromRGB(255, 0, 0) -- Cor da linha (vermelha)
    linha.Position = (player1.Character.HumanoidRootPart.Position + player2.Character.HumanoidRootPart.Position) / 2
    linha.Parent = game.Workspace

    -- Rotaciona a linha para conectar os jogadores
    local direction = (player2.Character.HumanoidRootPart.Position - player1.Character.HumanoidRootPart.Position).unit
    linha.CFrame = CFrame.new(linha.Position, player2.Character.HumanoidRootPart.Position)
    return linha
end

-- Função para criar o nome acima da cabeça do jogador
local function criarNomeAcimaDaCabeca(player)
    -- Cria um BillboardGui para mostrar o nome acima da cabeça
    local nomeGui = Instance.new("BillboardGui")
    nomeGui.Adornee = player.Character.Head
    nomeGui.Size = UDim2.new(0, 100, 0, 50)
    nomeGui.StudsOffset = Vector3.new(0, 3, 0) -- Desloca o nome para cima
    nomeGui.Parent = player.Character

    -- Cria um TextLabel para mostrar o nome
    local texto = Instance.new("TextLabel")
    texto.Text = player.Name
    texto.TextColor3 = Color3.fromRGB(255, 0, 0) -- Cor vermelha
    texto.BackgroundTransparency = 1
    texto.Size = UDim2.new(1, 0, 1, 0)
    texto.Parent = nomeGui
end

-- Função para monitorar a desconexão dos jogadores
local function monitorarDesconexao(player, linhas, nomes)
    player.AncestryChanged:Connect(function(_, parent)
        if not parent then
            -- Remove a linha e o nome quando o jogador desconectar
            if linhas[player] then
                linhas[player]:Destroy()
                linhas[player] = nil
            end
            if nomes[player] then
                nomes[player]:Destroy()
                nomes[player] = nil
            end
        end
    end)
end

-- Função principal para gerenciar os jogadores
local function gerenciarJogadores()
    local linhas = {} -- Tabela para armazenar as linhas
    local nomes = {}  -- Tabela para armazenar os nomes

    -- Quando um novo jogador entrar no jogo
    game.Players.PlayerAdded:Connect(function(player)
        -- Aguarda até o personagem do jogador estar carregado
        player.CharacterAdded:Connect(function(character)
            -- Cria o nome acima da cabeça
            criarNomeAcimaDaCabeca(player)
            
            -- Cria linhas entre o jogador e os outros jogadores
            for _, otherPlayer in pairs(game.Players:GetPlayers()) do
                if otherPlayer ~= player then
                    -- Cria a linha entre os jogadores
                    linhas[player] = criarLinhaEntreJogadores(player, otherPlayer)
                end
            end

            -- Monitorar a desconexão do jogador
            monitorarDesconexao(player, linhas, nomes)
        end)
    end)

    -- Monitorar jogadores que já estão no jogo ao iniciar
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character then
            criarNomeAcimaDaCabeca(player)
            for _, otherPlayer in pairs(game.Players:GetPlayers()) do
                if otherPlayer ~= player then
                    linhas[player] = criarLinhaEntreJogadores(player, otherPlayer)
                end
            end
            monitorarDesconexao(player, linhas, nomes)
        end
    end
end

-- Chama a função principal para gerenciar os jogadores
gerenciarJogadores()
