function love.load()
    love.window.setTitle("Monster Hub")
    love.window.setMode(800, 600)

    fonte = love.graphics.newFont(32)
    fontePequena = love.graphics.newFont(18)
    fonteMini = love.graphics.newFont(14)

    -- Botão abrir/fechar
    botaoAberto = true
    botaoX = 800 - 120 - 20
    botaoY = 20
    botaoLargura = 120
    botaoAltura = 50

    -- Quadrado alvo
    quadLargura = 200
    quadAltura = 330
    quadRaioBorda = 25
    quadX = (800 - quadLargura)/2
    quadY = (600 - quadAltura)/2

    -- Quadrado animado (para transição)
    quadAnimLargura = 0
    quadAnimAltura = 0
    quadAnimRaio = 0
    quadAnimSpeed = 10 -- velocidade da animação

    -- Botão "Início"
    inicioLargura = 120
    inicioAltura = 30
    inicioX = quadX + 10
    inicioY = quadY + 10
    opcoesVisiveis = false

    -- Opção Xray
    xrayAtivado = false
    xrayAnim = 0

    -- Slider velocidade
    sliderX = inicioX
    sliderY = inicioY + 80
    sliderLargura = 160
    sliderAltura = 10
    velocidadeAtual = 0
    arrastandoSlider = false

    -- Botão ativar velocidade
    botaoVelocidadeX = inicioX
    botaoVelocidadeY = sliderY + 30
    botaoVelocidadeLargura = 160
    botaoVelocidadeAltura = 30
    velocidadeAtivada = 0

    -- Círculo minimizado
    circRaio = 50
    circX = 100
    circY = 100
    arrastando = false

    -- Dragão
    dragaoImg = love.graphics.newImage("dragon.png")

    -- Inimigos e paredes
    inimigos = {
        {x = quadX + 50, y = quadY + 50, raio = 10},
        {x = quadX + 150, y = quadY + 80, raio = 10},
        {x = quadX + 120, y = quadY + 150, raio = 10}
    }
    paredes = {
        {x = quadX + 30, y = quadY + 40, w = 50, h = 100},
        {x = quadX + 100, y = quadY + 20, w = 60, h = 50},
        {x = quadX + 140, y = quadY + 100, w = 40, h = 80}
    }
end

function love.update(dt)
    -- Animação Xray
    if xrayAtivado then
        xrayAnim = xrayAnim + dt*5
        if xrayAnim > 1 then xrayAnim = 0 end
    end

    -- Transição quadrado ↔ círculo
    local targetL, targetA, targetRaio
    if botaoAberto then
        targetL = quadLargura
        targetA = quadAltura
        targetRaio = quadRaioBorda
    else
        targetL = circRaio*2
        targetA = circRaio*2
        targetRaio = circRaio
    end

    quadAnimLargura = quadAnimLargura + (targetL - quadAnimLargura) * dt * quadAnimSpeed
    quadAnimAltura = quadAnimAltura + (targetA - quadAnimAltura) * dt * quadAnimSpeed
    quadAnimRaio = quadAnimRaio + (targetRaio - quadAnimRaio) * dt * quadAnimSpeed
end

function love.draw()
    love.graphics.setFont(fonte)
    love.graphics.setColor(1,1,1)
    love.graphics.print("Monster Hub", 20, 20)

    -- Botão abrir/fechar
    love.graphics.setColor(botaoAberto and {0,1,0} or {1,0,0})
    love.graphics.rectangle("fill", botaoX, botaoY, botaoLargura, botaoAltura, 10, 10)
    love.graphics.setColor(1,1,1)
    love.graphics.print(botaoAberto and "Aberto" or "Fechado", botaoX+10, botaoY+10)

    -- Quadrado/círculo animado
    local cx = (800 - quadAnimLargura)/2
    local cy = (600 - quadAnimAltura)/2

    love.graphics.setColor(0,0,0,0.4)
    love.graphics.rectangle("fill", cx+8, cy+8, quadAnimLargura, quadAnimAltura, quadAnimRaio, quadAnimRaio)
    love.graphics.setColor(1,0,0)
    love.graphics.rectangle("fill", cx, cy, quadAnimLargura, quadAnimAltura, quadAnimRaio, quadAnimRaio)

    if botaoAberto or quadAnimLargura > 1 then
        local mouseX, mouseY = love.mouse.getPosition()

        -- Botão Início hover
        local hoverInicio = mouseX >= inicioX and mouseX <= inicioX+inicioLargura and mouseY >= inicioY and mouseY <= inicioY+inicioAltura
        love.graphics.setColor(hoverInicio and {0.3,0.7,1} or {0.2,0.6,1})
        love.graphics.rectangle("fill", inicioX, inicioY, inicioLargura, inicioAltura, 5, 5)
        love.graphics.setColor(1,1,1)
        love.graphics.setFont(fontePequena)
        love.graphics.print("Início", inicioX+10, inicioY+5)

        -- Xray
        if opcoesVisiveis then
            local opcY = inicioY+40
            love.graphics.setColor(0.1,0.1,0.1)
            love.graphics.rectangle("fill", inicioX, opcY, inicioLargura, 30, 5,5)
            love.graphics.setColor(xrayAtivado and {0,0.8 + 0.2*math.sin(xrayAnim*math.pi*2),1} or {1,1,1})
            love.graphics.print("Xray", inicioX+10, opcY+5)
            love.graphics.setFont(fonteMini)
            love.graphics.setColor(1,1,1,0.4)
            love.graphics.print("Vê oponentes através da parede", inicioX, opcY+25)
        end

        -- Slider velocidade
        love.graphics.setFont(fontePequena)
        love.graphics.setColor(1,1,1)
        love.graphics.print("Velocidade: "..velocidadeAtual, sliderX, sliderY-20)
        love.graphics.setColor(0.3,0.3,0.3)
        love.graphics.rectangle("fill", sliderX, sliderY, sliderLargura, sliderAltura, 5,5)
        local larguraPreenchida = (velocidadeAtual/500)*sliderLargura
        local corSlider = arrastandoSlider and {0.2,0.6,1} or {0.2,0.8,0.2}
        love.graphics.setColor(corSlider)
        love.graphics.rectangle("fill", sliderX, sliderY, larguraPreenchida, sliderAltura, 5,5)
        local posIndicador = sliderX + larguraPreenchida
        love.graphics.setColor(arrastandoSlider and {1,1,1} or {0.9,0.9,0.9})
        love.graphics.circle("fill", posIndicador, sliderY + sliderAltura/2, arrastandoSlider and 10 or 8)

        -- Botão ativar velocidade hover
        local hoverBotao = mouseX >= botaoVelocidadeX and mouseX <= botaoVelocidadeX+botaoVelocidadeLargura and
                           mouseY >= botaoVelocidadeY and mouseY <= botaoVelocidadeY+botaoVelocidadeAltura
        love.graphics.setColor(hoverBotao and {0.3,0.9,0.3} or {0.2,0.8,0.2})
        love.graphics.rectangle("fill", botaoVelocidadeX, botaoVelocidadeY, botaoVelocidadeLargura, botaoVelocidadeAltura, 5,5)
        love.graphics.setColor(1,1,1)
        love.graphics.print("Ativar Velocidade", botaoVelocidadeX+10, botaoVelocidadeY+5)
        -- Texto explicativo
        love.graphics.setFont(fonteMini)
        love.graphics.setColor(1,1,1,0.4)
        love.graphics.print("Ativa a velocidade de acordo com o número", botaoVelocidadeX, botaoVelocidadeY + botaoVelocidadeAltura + 5)

        -- Paredes
        for _, p in ipairs(paredes) do
            love.graphics.setColor(0.5,0.5,0.5)
            love.graphics.rectangle("fill", p.x, p.y, p.w, p.h)
        end

        -- Inimigos
        for _, i in ipairs(inimigos) do
            if xrayAtivado then
                love.graphics.setColor(0,1,1,0.7)
                love.graphics.circle("fill", i.x, i.y, i.raio+2)
            else
                local bloqueado = false
                for _, p in ipairs(paredes) do
                    if i.x > p.x and i.x < p.x+p.w and i.y > p.y and i.y < p.y+p.h then bloqueado=true end
                end
                if not bloqueado then
                    love.graphics.setColor(0,1,0)
                    love.graphics.circle("fill", i.x, i.y, i.raio)
                end
            end
        end
    else
        -- Círculo dragão
        love.graphics.setColor(0,0,0,0.4)
        love.graphics.circle("fill", circX+4, circY+4, circRaio)
        love.graphics.setColor(1,1,1)
        love.graphics.draw(dragaoImg, circX - dragaoImg:getWidth()/2, circY - dragaoImg:getHeight()/2)
    end
end

function love.mousepressed(mx, my, button)
    if button==1 then
        -- abrir/fechar
        if mx >= botaoX and mx <= botaoX+botaoLargura and my >= botaoY and my <= botaoY+botaoAltura then
            botaoAberto = not botaoAberto
        end

        if botaoAberto then
            -- botão "Início"
            if mx >= inicioX and mx <= inicioX+inicioLargura and my >= inicioY and my <= inicioY+inicioAltura then
                opcoesVisiveis = not opcoesVisiveis
            end

            -- Xray
            local opcY = inicioY + 40
            if opcoesVisiveis and mx >= inicioX and mx <= inicioX+inicioLargura and my >= opcY and my <= opcY+30 then
                xrayAtivado = not xrayAtivado
            end

            -- Slider
            if mx >= sliderX and mx <= sliderX + sliderLargura and my >= sliderY -5 and my <= sliderY + sliderAltura +5 then
                arrastandoSlider = true
            end

            -- Botão ativar velocidade
            if mx >= botaoVelocidadeX and mx <= botaoVelocidadeX+botaoVelocidadeLargura and
               my >= botaoVelocidadeY and my <= botaoVelocidadeY + botaoVelocidadeAltura then
                velocidadeAtivada = velocidadeAtual
                print("Velocidade ativada:", velocidadeAtivada)
            end
        end

        -- Arrastar dragão
        if not botaoAberto then
            local dx = mx - circX
            local dy = my - circY
            if dx*dx + dy*dy <= circRaio*circRaio then
                arrastando = true
                offsetX = dx
                offsetY = dy
            end
        end
    end
end

function love.mousereleased(mx, my, button)
    if button==1 then
        arrastando = false
        arrastandoSlider = false
    end
end

function love.mousemoved(mx, my, dx, dy)
    if arrastando then
        circX = mx - offsetX
        circY = my - offsetY
    end
    if arrastandoSlider then
        if mx < sliderX then mx = sliderX end
        if mx > sliderX + sliderLargura then mx = sliderX + sliderLargura end
        velocidadeAtual = math.floor(((mx - sliderX) / sliderLargura) * 500)
    end
end
