-- =============================================================================
-- GRAVADOR PRO CONSEGUIDOR DE ROTAS + GERENCIADOR WORKSPACE (1:1 CORRIGIDO)
-- =============================================================================
if game.CoreGui:FindFirstChild("GravadorAutoExecuteGUI") then 
    game.CoreGui.GravadorAutoExecuteGUI:Destroy() 
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer

local gravando = false
local dadosTrajeto = {} 
local tempoDecorrido = 0
local ultimaPosicaoSalva = Vector3.new(0, 0, 0)
local executandoAuto = false

-- Garantir que a pasta das rotas exista no Workspace do seu Executor
if makefolder then
    pcall(function() makefolder("MinhasRotasParkour") end)
end

-- Interface Visual Expandida para Listagem
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "GravadorAutoExecuteGUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 320) -- Aumentado para caber a lista
frame.Position = UDim2.new(0, 15, 0, 140)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
frame.Active = true frame.Draggable = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

-- Título do Painel
local txtTitulo = Instance.new("TextLabel", frame)
txtTitulo.Size = UDim2.new(1, 0, 0, 25)
txtTitulo.BackgroundTransparency = 1
txtTitulo.Text = "FEITO POR BY KLOVZZADA"
txtTitulo.TextColor3 = Color3.fromRGB(255, 255, 255)
txtTitulo.Font = Enum.Font.SourceSansBold
txtTitulo.TextSize = 12

-- Container com Scroll para listar as rotas salvas
local scrollLista = Instance.new("ScrollingFrame", frame)
scrollLista.Size = UDim2.new(0, 180, 0, 120)
scrollLista.Position = UDim2.new(0, 10, 0, 165)
scrollLista.BackgroundTransparency = 0.9
scrollLista.BackgroundColor3 = Color3.fromRGB(0,0,0)
scrollLista.BorderSizePixel = 0
scrollLista.CanvasSize = UDim2.new(0, 0, 0, 0)
local listaLayout = Instance.new("UIListLayout", scrollLista)
listaLayout.Padding = UDim.new(0, 4)

local function criarBotao(texto, pos, cor, clique)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 180, 0, 32) btn.Position = pos
    btn.BackgroundColor3 = cor btn.Text = texto
    btn.TextColor3 = Color3.fromRGB(255, 255, 255) btn.Font = Enum.Font.SourceSansBold btn.TextSize = 13
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    btn.MouseButton1Click:Connect(clique)
    return btn
end

local btnRec, btnAuto, btnSave, lblStatus

-- Função de movimentação compartilhada (Memória ou Arquivo)
local function executarMovimento(tabelaFrames, botaoAlvo)
    if #tabelaFrames == 0 or executandoAuto then return end
    
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    
    if root and humanoid then
        executandoAuto = true
        local corOriginal = botaoAlvo.BackgroundColor3
        local textoOriginal = botaoAlvo.Text
        
        botaoAlvo.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
        botaoAlvo.Text = "EXECUTANDO..."
        lblStatus.Text = "Física 1:1 ativa..."
        
        root.CFrame = tabelaFrames[1].CFrame
        task.wait(0.1)
        
        local tempoInicio = os.clock()
        local ponteiroFrame = 1
        local conexao
        
        conexao = RunService.Heartbeat:Connect(function()
            local tempoAtual = os.clock() - tempoInicio
            
            while ponteiroFrame < #tabelaFrames and tabelaFrames[ponteiroFrame + 1].tempo <= tempoAtual do
                ponteiroFrame = ponteiroFrame + 1
            end
            
            if ponteiroFrame >= #tabelaFrames or not executandoAuto then
                conexao:Disconnect()
                executandoAuto = false
                botaoAlvo.BackgroundColor3 = corOriginal
                botaoAlvo.Text = textoOriginal
                lblStatus.Text = "Finalizado!"
                return
            end
            
            local fAtual = tabelaFrames[ponteiroFrame]
            local fProximo = tabelaFrames[ponteiroFrame + 1]
            local deltaTotal = fProximo.tempo - fAtual.tempo
            local alfa = deltaTotal > 0 and (tempoAtual - fAtual.tempo) / deltaTotal or 0
            
            root.CFrame = fAtual.CFrame:Lerp(fProximo.CFrame, math.clamp(alfa, 0, 1))
            if fAtual.pulando then humanoid.Jump = true end
        end)
    end
end

-- Atualiza a lista de arquivos salvos abaixo no painel
local function atualizarListaVisual()
    for _, child in ipairs(scrollLista:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end
    
    if not listfiles then return end
    
    local sucesso, arquivos = pcall(function() return listfiles("MinhasRotasParkour") end)
    if sucesso and arquivos then
        local contagem = 0
        for _, caminho in ipairs(arquivos) do
            local nomeArquivo = caminho:match("([^/^\\]+)$") or caminho
            if nomeArquivo:sub(-5) == ".json" then
                contagem = contagem + 1
                
                local btnRota = Instance.new("TextButton", scrollLista)
                btnRota.Size = UDim2.new(1, 0, 0, 25)
                btnRota.BackgroundColor3 = Color3.fromRGB(45, 45, 50)
                btnRota.Text = "▶ " .. nomeArquivo:gsub(".json", "")
                btnRota.TextColor3 = Color3.fromRGB(230, 230, 230)
                btnRota.Font = Enum.Font.SourceSansBold
                btnRota.TextSize = 12
                Instance.new("UICorner", btnRota).CornerRadius = UDim.new(0, 4)
                
                btnRota.MouseButton1Click:Connect(function()
                    if executandoAuto then executandoAuto = false task.wait(0.1) end
                    
                    local conteudo = readfile(caminho)
                    local dadosConvertidos = HttpService:JSONDecode(conteudo)
                    
                    -- Reconstrói os CFrames originais vindos do arquivo salvo
                    local tabelaReconstruida = {}
                    for _, f in ipairs(dadosConvertidos) do
                        table.insert(tabelaReconstruida, {
                            tempo = f.tempo,
                            CFrame = CFrame.new(unpack(f.cf)),
                            pulando = f.pulando
                        })
                    end
                    
                    executarMovimento(tabelaReconstruida, btnRota)
                end)
            end
        end
        scrollLista.CanvasSize = UDim2.new(0, 0, 0, contagem * 29)
    end
end

-- Botão 1: Gravar Rota
btnRec = criarBotao("● GRAVAR(faça o parkour)", UDim2.new(0, 10, 0, 30), Color3.fromRGB(220, 60, 60), function()
    if executandoAuto then executandoAuto = false task.wait(0.1) end
    gravando = not gravando
    if gravando then
        dadosTrajeto = {} tempoDecorrido = 0
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then ultimaPosicaoSalva = char.HumanoidRootPart.Position end
        btnRec.Text = "■ PARAR DE GRAVA" btnRec.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        btnAuto.Visible = false btnSave.Visible = false
        lblStatus.Text = "Gravando passos..."
    else
        btnRec.Text = "● GRAVAR ROTA REAL" btnRec.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
        lblStatus.Text = "Captura salva em cache!"
        btnAuto.Visible = true btnSave.Visible = true
    end
end)

-- Botão 2: Auto Execute (Teste Rápido)
btnAuto = criarBotao("⚡ VEJA COMO ESTA EXECUTE", UDim2.new(0, 10, 0, 68), Color3.fromRGB(200, 150, 0), function()
    if executandoAuto then executandoAuto = false else executarMovimento(dadosTrajeto, btnAuto) end
end)
btnAuto.Visible = false

-- Botão 3: Salvar direto para a Pasta com Nome Dinâmico
btnSave = criarBotao("⬇ DOWNLOAD WORKSPACE", UDim2.new(0, 10, 0, 106), Color3.fromRGB(0, 120, 200), function()
    if #dadosTrajeto == 0 then lblStatus.Text = "Erro: Sem dados!" return end
    if not writefile then lblStatus.Text = "Executor sem writefile!" return end
    
    -- Gera um nome único incremental ex: Rota_1, Rota_2 automaticamente
    local numeroRota = 1
    if listfiles then
        local logs = listfiles("MinhasRotasParkour")
        for _, f in ipairs(logs) do numeroRota = numeroRota + 1 end
    end
    
    local nomeFinal = "MinhasRotasParkour/Rota_" .. numeroRota .. ".json"
    
    -- Simplifica os dados para salvar perfeitamente como tabela leve
    local tabelaLeve = {}
    for _, frame in ipairs(dadosTrajeto) do
        table.insert(tabelaLeve, {
            tempo = frame.tempo,
            cf = {frame.CFrame:GetComponents()},
            pulando = frame.pulando
        })
    end
    
    pcall(function()
        writefile(nomeFinal, HttpService:JSONEncode(tabelaLeve))
    end)
    
    lblStatus.Text = "Salvo: Rota_" .. numeroRota
    atualizarListaVisual()
end)
btnSave.Visible = false

-- Status Text
lblStatus = Instance.new("TextLabel", frame)
lblStatus.Size = UDim2.new(0, 180, 0, 20) lblStatus.Position = UDim2.new(0, 10, 0, 142)
lblStatus.BackgroundTransparency = 1 lblStatus.Text = "Status: Aguardando..."
lblStatus.TextColor3 = Color3.fromRGB(180, 180, 180) lblStatus.TextSize = 11 lblStatus.Font = Enum.Font.SourceSans

-- Monitor de física de alta precisão
RunService.Heartbeat:Connect(function(dt)
    if gravando then
        tempoDecorrido = tempoDecorrido + dt
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local humanoid = char and char:FindFirstChildOfClass("Humanoid")
        
        if root and humanoid then
            local posicaoAtual = root.Position
            local estaPulando = (humanoid.FloorMaterial == Enum.Material.Air and root.Velocity.Y > 1)
            
            if (posicaoAtual - ultimaPosicaoSalva).Magnitude > 0.25 or estaPulando then
                table.insert(dadosTrajeto, {
                    tempo = tempoDecorrido,
                    CFrame = root.CFrame,
                    pulando = estaPulando
                })
                ultimaPosicaoSalva = posicaoAtual
                lblStatus.Text = "Pontos em cache: " .. #dadosTrajeto
            end
        end
    end
end)

-- Inicializa a lista carregando o que já estiver salvo na pasta
atualizarListaVisual()
