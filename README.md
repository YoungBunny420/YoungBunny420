- 👋 Hi, I’m @YoungBunny420
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- -- Improved ForcedWorkInkUI LocalScript
-- Features:
--  - Uses UserInputService-based dragging (frame.Draggable removed).
--  - Tries to open the browser safely, falls back to copying to clipboard (if available),
--    or shows a short instruction to copy manually.
--  - Close button shows a confirmation prompt.
--  - Non-editable link display with Copy / Open buttons and small feedback messages.
--  - Minimal, easily-customizable styling and sizes.
--
-- Note: Replace LINK_URL if you want another URL. This script does NOT force or require
-- visiting external sites â€” it only offers to open or copy the link.

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local LINK_URL = "https://work.ink/2a0Q/police-simulator-script" -- set/replace as needed

-- Helpers
local function safeOpenUrl(url)
	-- Try GuiService:OpenBrowserWindow first (may be restricted)
	local ok, err = pcall(function()
		-- GuiService.OpenBrowserWindow can throw in some environments
		GuiService:OpenBrowserWindow(url)
	end)
	return ok, err
end

local function tryCopyToClipboard(text)
	-- Some environments expose setclipboard; wrap in pcall
	local ok, err = pcall(function()
		-- setclipboard might exist in some exploits / Studio environments
		if setclipboard then
			setclipboard(text)
		-- In some Roblox environments there's a UserInputService:SetClipboard (rare); try that
		elseif UserInputService.SetClipboard then
			UserInputService:SetClipboard(text)
		else
			error("No clipboard function available")
		end
	end)
	return ok, err
end

local function showTransientMessage(parent, text, duration)
	duration = duration or 2.0
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0.9, 0, 0, 28)
	label.Position = UDim2.new(0.05, 0, 0.78, 0)
	label.BackgroundTransparency = 0.05
	label.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	label.TextColor3 = Color3.fromRGB(255, 255, 255)
	label.Text = text
	label.TextScaled = true
	label.Font = Enum.Font.Gotham
	label.AnchorPoint = Vector2.new(0, 0)
	label.Parent = parent
	label.ZIndex = 50
	local corner = Instance.new("UICorner", label)
	corner.CornerRadius = UDim.new(0, 8)

	local tweenInfo = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
	TweenService:Create(label, tweenInfo, {BackgroundTransparency = 0}):Play()

	delay(duration, function()
		TweenService:Create(label, tweenInfo, {BackgroundTransparency = 1}):Play()
		wait(0.18)
		if label and label.Parent then
			label:Destroy()
		end
	end)
end

-- Create ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "ForcedWorkInkUI"
gui.ResetOnSpawn = false
gui.DisplayOrder = 50
gui.Parent = playerGui

-- Main Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 420, 0, 200)
frame.Position = UDim2.new(0.5, -210, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Parent = gui
frame.Active = true -- needed for input capture when dragging

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Title bar
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundTransparency = 1
titleBar.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -80, 1, 0)
title.Position = UDim2.new(0, 12, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Police Simulator Script"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = titleBar

-- Close Button (with confirm)
local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 36, 0, 28)
close.Position = UDim2.new(1, -44, 0, 6)
close.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
close.Text = "X"
close.TextColor3 = Color3.fromRGB(255, 255, 255)
close.TextScaled = true
close.Font = Enum.Font.GothamBold
close.Parent = titleBar
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = close

-- Body contents: link label and buttons
local body = Instance.new("Frame")
body.Size = UDim2.new(1, -20, 1, -58)
body.Position = UDim2.new(0, 10, 0, 42)
body.BackgroundTransparency = 1
body.Parent = frame

-- Link display (non-editable)
local linkLabel = Instance.new("TextLabel")
linkLabel.Size = UDim2.new(1, -120, 0, 36)
linkLabel.Position = UDim2.new(0, 0, 0, 6)
linkLabel.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
linkLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
linkLabel.Text = LINK_URL
linkLabel.TextScaled = false
linkLabel.TextTruncate = Enum.TextTruncate.AtEnd
linkLabel.Font = Enum.Font.Gotham
linkLabel.TextXAlignment = Enum.TextXAlignment.Left
linkLabel.TextYAlignment = Enum.TextYAlignment.Center
linkLabel.Parent = body
local linkCorner = Instance.new("UICorner", linkLabel)
linkCorner.CornerRadius = UDim.new(0, 8)
local linkPadding = Instance.new("UIPadding", linkLabel)
linkPadding.PaddingLeft = UDim.new(0, 8)

-- Buttons: Open and Copy
local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0, 52, 0, 36)
openBtn.Position = UDim2.new(1, -52, 0, 6)
openBtn.AnchorPoint = Vector2.new(1, 0)
openBtn.BackgroundColor3 = Color3.fromRGB(60, 130, 200)
openBtn.Text = "Open"
openBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
openBtn.Font = Enum.Font.GothamBold
openBtn.TextScaled = true
openBtn.Parent = body
local openCorner = Instance.new("UICorner", openBtn)
openCorner.CornerRadius = UDim.new(0, 8)

local copyBtn = Instance.new("TextButton")
copyBtn.Size = UDim2.new(0, 60, 0, 36)
copyBtn.Position = UDim2.new(1, -120, 0, 6)
copyBtn.AnchorPoint = Vector2.new(1, 0)
copyBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
copyBtn.Text = "Copy"
copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
copyBtn.Font = Enum.Font.GothamBold
copyBtn.TextScaled = true
copyBtn.Parent = body
local copyCorner = Instance.new("UICorner", copyBtn)
copyCorner.CornerRadius = UDim.new(0, 8)

-- Short description/instruction
local info = Instance.new("TextLabel")
info.Size = UDim2.new(1, 0, 0, 84)
info.Position = UDim2.new(0, 0, 0, 52)
info.BackgroundTransparency = 1
info.TextColor3 = Color3.fromRGB(200, 200, 200)
info.Text = "Press Open to try to open the link in your browser. If that fails, use Copy then paste the link into your browser."
info.TextWrapped = true
info.TextScaled = true
info.Font = Enum.Font.Gotham
info.TextYAlignment = Enum.TextYAlignment.Top
info.Parent = body

-- Close confirmation modal (hidden by default)
local confirmFrame = Instance.new("Frame")
confirmFrame.Size = UDim2.new(0, 300, 0, 120)
confirmFrame.Position = UDim2.new(0.5, -150, 0.5, -60)
confirmFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
confirmFrame.BorderSizePixel = 0
confirmFrame.Visible = false
confirmFrame.ZIndex = 60
confirmFrame.Parent = gui
local confirmCorner = Instance.new("UICorner", confirmFrame)
confirmCorner.CornerRadius = UDim.new(0, 10)

local confirmText = Instance.new("TextLabel", confirmFrame)
confirmText.Size = UDim2.new(0.9, 0, 0, 56)
confirmText.Position = UDim2.new(0.05, 0, 0.12, 0)

<!---
YoungBunny420/YoungBunny420 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
