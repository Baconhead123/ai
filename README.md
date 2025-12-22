local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")

-- 免费无注册AI接口配置（慕名API，无需密钥/注册）
local FreeAI = {
    API_URL = "https://api.suol.cc/v1/ai.php",
    Personality = "你是精通Roblox脚本开发的助手，说话简洁，能生成Lua代码，响应准确实用"
}

-- 设置AI人设
function FreeAI:SetPersonality(newPersona)
    self.Personality = newPersona
end

-- 调用免费AI接口获取响应（带人设上下文）
function FreeAI:GetResponse(userMsg)
    local fullMsg = string.format("[人设]%s\n[用户需求]%s", self.Personality, userMsg)
    local requestUrl = string.format("%s?msg=%s", self.API_URL, HttpService:UrlEncode(fullMsg))
    
    local success, response = pcall(function()
        return HttpService:GetAsync(requestUrl, false)
    end)
    
    if success then
        local data = HttpService:JSONDecode(response)
        return data.content or "AI回复失败，请重试~"
    else
        return "网络异常，无法连接AI，请稍后再试"
    end
end

-- 创建微信风格UI
local function CreateWeChatUI()
    local Player = Players.LocalPlayer
    local PlayerGui = Player:WaitForChild("PlayerGui")
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "FreeAIChat"
    ScreenGui.Parent = PlayerGui
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    -- 主窗口
    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 420, 0, 650)
    MainFrame.Position = UDim2.new(0.5, -210, 0.5, -325)
    MainFrame.BackgroundColor3 = Color3.new(0.95, 0.95, 0.95)
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Parent = ScreenGui

    -- 标题栏
    local TitleBar = Instance.new("Frame")
    TitleBar.Size = UDim2.new(1, 0, 0, 45)
    TitleBar.BackgroundColor3 = Color3.new(0.08, 0.63, 0.27)
    TitleBar.Parent = MainFrame

    local TitleText = Instance.new("TextLabel")
    TitleText.Size = UDim2.new(1, 0, 1, 0)
    TitleText.BackgroundTransparency = 1
    TitleText.Text = "免费Roblox AI聊天助手"
    TitleText.TextColor3 = Color3.new(1, 1, 1)
    TitleText.Font = Enum.Font.SourceSansBold
    TitleText.TextSize = 19
    TitleText.Parent = TitleBar

    -- 人设按钮
    local PersonaBtn = Instance.new("TextButton")
    PersonaBtn.Size = UDim2.new(0, 90, 0, 32)
    PersonaBtn.Position = UDim2.new(1, -100, 0.5, -16)
    PersonaBtn.BackgroundColor3 = Color3.new(0.18, 0.72, 0.38)
    PersonaBtn.Text = "修改人设"
    PersonaBtn.TextColor3 = Color3.new(1, 1, 1)
    PersonaBtn.TextSize = 15
    PersonaBtn.BorderSizePixel = 0
    PersonaBtn.Parent = TitleBar

    -- 聊天容器
    local ChatContainer = Instance.new("ScrollingFrame")
    ChatContainer.Size = UDim2.new(1, -20, 1, -110)
    ChatContainer.Position = UDim2.new(0, 10, 0, 45)
    ChatContainer.BackgroundTransparency = 1
    ChatContainer.ScrollBarThickness = 6
    ChatContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
    ChatContainer.Parent = MainFrame

    -- 输入区域
    local InputFrame = Instance.new("Frame")
    InputFrame.Size = UDim2.new(1, -20, 0, 45)
    InputFrame.Position = UDim2.new(0, 10, 1, -60)
    InputFrame.BackgroundColor3 = Color3.new(1, 1, 1)
    InputFrame.Parent = MainFrame

    local InputBox = Instance.new("TextBox")
    InputBox.Size = UDim2.new(1, -75, 1, 0)
    InputBox.BackgroundColor3 = Color3.new(0.97, 0.97, 0.97)
    InputBox.PlaceholderText = "输入需求（写脚本/聊天/改人设）..."
    InputBox.TextSize = 15
    InputBox.ClearTextOnFocus = false
    InputBox.BorderSizePixel = 0
    InputBox.Parent = InputFrame

    local SendBtn = Instance.new("TextButton")
    SendBtn.Size = UDim2.new(0, 65, 1, 0)
    SendBtn.Position = UDim2.new(1, -65, 0, 0)
    SendBtn.BackgroundColor3 = Color3.new(0.08, 0.63, 0.27)
    SendBtn.Text = "发送"
    SendBtn.TextColor3 = Color3.new(1, 1, 1)
    SendBtn.TextSize = 15
    SendBtn.BorderSizePixel = 0
    SendBtn.Parent = InputFrame

    -- 添加聊天气泡
    local function AddChatBubble(isUser, content)
        local Bubble = Instance.new("Frame")
        Bubble.BackgroundTransparency = 1
        Bubble.Parent = ChatContainer

        local TextLabel = Instance.new("TextLabel")
        TextLabel.Size = UDim2.new(0, 310, 0, 0)
        TextLabel.BackgroundColor3 = isUser and Color3.new(0.32, 0.82, 0.32) or Color3.new(1, 1, 1)
        TextLabel.Text = content
        TextLabel.TextColor3 = isUser and Color3.new(1, 1, 1) or Color3.new(0, 0, 0)
        TextLabel.TextSize = 15
        TextLabel.TextWrapped = true
        TextLabel.Font = Enum.Font.SourceSans
        TextLabel.Padding = UDim.new(0, 10)
        TextLabel.Parent = Bubble

        -- 自动适配文本大小
        TextLabel.Size = UDim2.new(0, 310, 0, TextLabel.TextBounds.Y + 20)
        Bubble.Size = TextLabel.Size
        Bubble.Position = UDim2.new(
            isUser and 1 or 0,
            isUser and -320 or 10,
            0,
            ChatContainer.CanvasSize.Y.Offset + 15
        )

        -- 更新滚动条
        ChatContainer.CanvasSize = UDim2.new(0, 0, 0, ChatContainer.CanvasSize.Y.Offset + Bubble.Size.Y.Offset + 15)
        ChatContainer.CanvasPosition = Vector2.new(0, ChatContainer.CanvasSize.Y.Offset)
    end

    -- 发送消息逻辑
    local function SendMsg()
        local msg = InputBox.Text:trim()
        if msg == "" then return end
        AddChatBubble(true, msg)
        InputBox.Text = ""

        -- 处理人设修改
        if string.find(string.lower(msg), "人设") then
            local newPersona = string.gsub(msg, "人设", ""):trim()
            if newPersona ~= "" then
                FreeAI:SetPersonality(newPersona)
                AddChatBubble(false, "✅ 人设已更新：" .. newPersona)
                return
            end
        end

        -- 加载中提示
        AddChatBubble(false, "🤖 AI思考中...")
        local lastBubble = ChatContainer:GetChildren()[#ChatContainer:GetChildren()]

        -- 异步获取AI响应
        task.spawn(function()
            local response = FreeAI:GetResponse(msg)
            lastBubble:Destroy()
            AddChatBubble(false, response)
        end)
    end

    -- 绑定事件
    SendBtn.MouseButton1Click:Connect(SendMsg)
    InputBox.FocusLost:Connect(function(enterPressed)
        if enterPressed then SendMsg() end
    end)
    PersonaBtn.MouseButton1Click:Connect(function()
        InputBox.Text = "人设："
        InputBox:CaptureFocus()
    end)

    -- 窗口拖拽
    local dragging, dragStart, startPos = false, Vector2.new(), Vector2.new()
    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    -- 初始欢迎消息
    AddChatBubble(false, "🎉 你好！我是免费AI助手，无需注册即可使用～\n支持：写Roblox脚本、自由聊天、自定义人设\n示例：\n- 写一个零件移动脚本\n- 人设：搞笑的编程大神\n- 怎么实现角色跳跃功能？")
end

-- 客户端启动
if RunService:IsClient() then
    CreateWeChatUI()
end

return FreeAI
