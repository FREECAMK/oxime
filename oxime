local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local gui = Instance.new("ScreenGui")
gui.Name = "PurpleMenu"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui
local blur = Instance.new("BlurEffect")
blur.Name = "MenuBlur"
blur.Size = 0
blur.Parent = Lighting
TweenService:Create(
    blur,
    TweenInfo.new(
        0.25,
        Enum.EasingStyle.Quad,
        Enum.EasingDirection.Out
    ),
    {
        Size = 10
    }
):Play()
local ghostLayer = Instance.new("Frame")
ghostLayer.Name = "GhostLayer"
ghostLayer.Size = UDim2.fromScale(1, 1)
ghostLayer.Position = UDim2.fromScale(0, 0)
ghostLayer.BackgroundTransparency = 1
ghostLayer.BorderSizePixel = 0
ghostLayer.ZIndex = 2
ghostLayer.Parent = gui
local wrapper = Instance.new("Frame")
wrapper.Name = "Background"
wrapper.Size = UDim2.fromOffset(680, 455)
wrapper.Position = UDim2.fromScale(0.5, 0.5)
wrapper.AnchorPoint = Vector2.new(0.5, 0.5)
wrapper.BackgroundColor3 = Color3.fromRGB(120, 40, 180)
wrapper.BackgroundTransparency = 0.78
wrapper.BorderSizePixel = 0
wrapper.ZIndex = 5
wrapper.Parent = gui
local wrapperCorner = Instance.new("UICorner")
wrapperCorner.CornerRadius = UDim.new(0, 20)
wrapperCorner.Parent = wrapper
local wrapperStroke = Instance.new("UIStroke")
wrapperStroke.Color = Color3.fromRGB(185, 90, 255)
wrapperStroke.Thickness = 1
wrapperStroke.Transparency = 0.8
wrapperStroke.Parent = wrapper
local menu = Instance.new("Frame")
menu.Name = "Menu"
menu.Size = UDim2.fromOffset(650, 425)
menu.Position = UDim2.fromScale(0.5, 0.5)
menu.AnchorPoint = Vector2.new(0.5, 0.5)
menu.BackgroundColor3 = Color3.fromRGB(123, 43, 181)
menu.BorderSizePixel = 0
menu.Active = true
menu.ZIndex = 10
menu.Parent = wrapper
local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 14)
menuCorner.Parent = menu
local menuStroke = Instance.new("UIStroke")
menuStroke.Color = Color3.fromRGB(205, 120, 255)
menuStroke.Thickness = 1
menuStroke.Transparency = 0.82
menuStroke.Parent = menu
local scale = Instance.new("UIScale")
scale.Scale = 1
scale.Parent = menu
local baseMenuSize = Vector2.new(650, 425)
local baseWrapperSize = Vector2.new(680, 455)
local dragging = false
local dragStart
local startPosition
local lastMousePosition
local lastTime
local targetX = 0
local targetY = 0
local currentX = 0
local currentY = 0
local velocityX = 0
local velocityY = 0
local targetRotation = 0
local currentRotation = 0
local targetScale = 1
local currentScale = 1
local targetStretchX = 1
local targetStretchY = 1
local currentStretchX = 1
local currentStretchY = 1
local bounce = 0.68
local wallPadding = 8
local overshootX = 0
local overshootY = 0
local overshootVelocityX = 0
local overshootVelocityY = 0
local impactSquashX = 1
local impactSquashY = 1
local ghosts = {}
local ghostCount = 7
for i = 1, ghostCount do
    local ghost = Instance.new("Frame")
    ghost.Name = "Ghost_" .. i
    ghost.AnchorPoint = Vector2.new(0.5, 0.5)
    ghost.Size = UDim2.fromOffset(650, 425)
    ghost.Position = UDim2.fromScale(0.5, 0.5)
    ghost.BackgroundColor3 = Color3.fromRGB(123, 43, 181)
    ghost.BackgroundTransparency = 1
    ghost.BorderSizePixel = 0
    ghost.ZIndex = 2
    ghost.Parent = ghostLayer
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 14)
    corner.Parent = ghost
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(205, 120, 255)
    stroke.Thickness = 1
    stroke.Transparency = 1
    stroke.Parent = ghost
    ghosts[i] = {
        frame = ghost,
        stroke = stroke,
        x = 0,
        y = 0,
        rotation = 0,
        alpha = 0,
        scale = 1
    }
end
local function updateGhosts()
    for i = ghostCount, 2, -1 do
        local previous = ghosts[i - 1]
        local current = ghosts[i]
        current.x = previous.x
        current.y = previous.y
        current.rotation = previous.rotation
        current.alpha = previous.alpha
        current.scale = previous.scale
    end
    local first = ghosts[1]
    first.x = currentX
    first.y = currentY
    first.rotation = currentRotation
    first.scale = currentScale
    first.alpha = 0.55
end
local function getScreenBounds()
    local camera = workspace.CurrentCamera
    if not camera then
        return 0, 0, 0, 0
    end
    local viewport = camera.ViewportSize
    local width =
        baseWrapperSize.X *
        currentScale *
        currentStretchX *
        impactSquashX
    local height =
        baseWrapperSize.Y *
        currentScale *
        currentStretchY *
        impactSquashY
    local halfWidth = width * 0.5
    local halfHeight = height * 0.5
    local minX =
        -viewport.X * 0.5 +
        halfWidth +
        wallPadding
    local maxX =
        viewport.X * 0.5 -
        halfWidth -
        wallPadding
    local minY =
        -viewport.Y * 0.5 +
        halfHeight +
        wallPadding
    local maxY =
        viewport.Y * 0.5 -
        halfHeight -
        wallPadding
    return minX, maxX, minY, maxY
end
local function triggerImpact(horizontal)
    if horizontal then
        impactSquashX = 0.92
        impactSquashY = 1.035
    else
        impactSquashX = 1.035
        impactSquashY = 0.92
    end
end
local function handleWalls()
    local minX, maxX, minY, maxY = getScreenBounds()
    if targetX < minX then
        targetX = minX
        if velocityX < 0 then
            local impactSpeed = math.abs(velocityX)
            velocityX =
                impactSpeed * bounce
            overshootX =
                -math.clamp(
                    impactSpeed * 0.006,
                    2,
                    9
                )
            overshootVelocityX =
                -impactSpeed * 0.006
            triggerImpact(true)
        end
    elseif targetX > maxX then
        targetX = maxX
        if velocityX > 0 then
            local impactSpeed = math.abs(velocityX)
            velocityX =
                -impactSpeed * bounce
            overshootX =
                math.clamp(
                    impactSpeed * 0.006,
                    2,
                    9
                )
            overshootVelocityX =
                impactSpeed * 0.006
            triggerImpact(true)
        end
    end
    if targetY < minY then
        targetY = minY
        if velocityY < 0 then
            local impactSpeed = math.abs(velocityY)
            velocityY =
                impactSpeed * bounce
            overshootY =
                -math.clamp(
                    impactSpeed * 0.006,
                    2,
                    9
                )
            overshootVelocityY =
                -impactSpeed * 0.006
            triggerImpact(false)
        end
    elseif targetY > maxY then
        targetY = maxY
        if velocityY > 0 then
            local impactSpeed = math.abs(velocityY)
            velocityY =
                -impactSpeed * bounce
            overshootY =
                math.clamp(
                    impactSpeed * 0.006,
                    2,
                    9
                )
            overshootVelocityY =
                impactSpeed * 0.006
            triggerImpact(false)
        end
    end
end
local content = Instance.new("Frame")
content.Name = "Content"
content.Size = UDim2.new(1, -82, 1, -30)
content.Position = UDim2.fromOffset(18, 15)
content.BackgroundTransparency = 1
content.BorderSizePixel = 0
content.ZIndex = 15
content.Parent = menu
local sectionTitle = Instance.new("TextLabel")
sectionTitle.Name = "SectionTitle"
sectionTitle.Size = UDim2.new(1, -20, 0, 36)
sectionTitle.Position = UDim2.fromOffset(10, 4)
sectionTitle.BackgroundTransparency = 1
sectionTitle.Text = "General"
sectionTitle.TextColor3 = Color3.fromRGB(245, 225, 255)
sectionTitle.TextSize = 22
sectionTitle.Font = Enum.Font.GothamMedium
sectionTitle.TextXAlignment = Enum.TextXAlignment.Left
sectionTitle.TextYAlignment = Enum.TextYAlignment.Center
sectionTitle.ZIndex = 16
sectionTitle.Parent = content
local sectionLine = Instance.new("Frame")
sectionLine.Name = "SectionLine"
sectionLine.Size = UDim2.new(1, -20, 0, 1)
sectionLine.Position = UDim2.fromOffset(10, 45)
sectionLine.BackgroundColor3 = Color3.fromRGB(205, 120, 255)
sectionLine.BackgroundTransparency = 0.78
sectionLine.BorderSizePixel = 0
sectionLine.ZIndex = 16
sectionLine.Parent = content
local sectionDescription = Instance.new("TextLabel")
sectionDescription.Name = "SectionDescription"
sectionDescription.Size = UDim2.new(1, -20, 0, 25)
sectionDescription.Position = UDim2.fromOffset(10, 50)
sectionDescription.BackgroundTransparency = 1
sectionDescription.Text = "General settings"
sectionDescription.TextColor3 = Color3.fromRGB(195, 160, 215)
sectionDescription.TextSize = 12
sectionDescription.Font = Enum.Font.Gotham
sectionDescription.TextXAlignment = Enum.TextXAlignment.Left
sectionDescription.TextYAlignment = Enum.TextYAlignment.Center
sectionDescription.ZIndex = 16
sectionDescription.Parent = content
local sectionData = {
    General = {
        title = "General",
        description = "General settings"
    },
    Visual = {
        title = "Visual",
        description = "Visual settings"
    },
    Misc = {
        title = "Misc",
        description = "Miscellaneous settings"
    }
}
local sectionAnimationId = 0
local function animateSection(name)
    local data = sectionData[name]
    if not data then
        return
    end
    sectionAnimationId += 1
    local animationId =
        sectionAnimationId
    local oldTitlePosition =
        UDim2.fromOffset(
            10,
            4
        )
    local oldDescriptionPosition =
        UDim2.fromOffset(
            10,
            50
        )
    TweenService:Create(
        sectionTitle,
        TweenInfo.new(
            0.12,
            Enum.EasingStyle.Quad,
            Enum.EasingDirection.In
        ),
        {
            TextTransparency = 1,
            Position =
                oldTitlePosition +
                UDim2.fromOffset(
                    8,
                    0
                )
        }
    ):Play()
    TweenService:Create(
        sectionDescription,
        TweenInfo.new(
            0.12,
            Enum.EasingStyle.Quad,
            Enum.EasingDirection.In
        ),
        {
            TextTransparency = 1,
            Position =
                oldDescriptionPosition +
                UDim2.fromOffset(
                    8,
                    0
                )
        }
    ):Play()
    task.delay(
        0.12,
        function()
            if animationId ~=
                sectionAnimationId then
                return
            end
            sectionTitle.Text =
                data.title
            sectionDescription.Text =
                data.description
            sectionTitle.Position =
                oldTitlePosition -
                UDim2.fromOffset(
                    8,
                    0
                )
            sectionDescription.Position =
                oldDescriptionPosition -
                UDim2.fromOffset(
                    8,
                    0
                )
            TweenService:Create(
                sectionTitle,
                TweenInfo.new(
                    0.22,
                    Enum.EasingStyle.Quart,
                    Enum.EasingDirection.Out
                ),
                {
                    TextTransparency = 0,
                    Position =
                        oldTitlePosition
                }
            ):Play()
            TweenService:Create(
                sectionDescription,
                TweenInfo.new(
                    0.22,
                    Enum.EasingStyle.Quart,
                    Enum.EasingDirection.Out
                ),
                {
                    TextTransparency = 0,
                    Position =
                        oldDescriptionPosition
                }
            ):Play()
        end
    )
end
local visualPanel = Instance.new("Frame")
visualPanel.Name = "VisualPanel"
visualPanel.Size = UDim2.new(1, -20, 0, 76)
visualPanel.Position = UDim2.fromOffset(10, 90)
visualPanel.BackgroundColor3 = Color3.fromRGB(104, 31, 151)
visualPanel.BackgroundTransparency = 0.3
visualPanel.BorderSizePixel = 0
visualPanel.Visible = false
visualPanel.ZIndex = 16
visualPanel.Parent = content
local visualPanelCorner =
    Instance.new("UICorner")
visualPanelCorner.CornerRadius =
    UDim.new(
        0,
        12
    )
visualPanelCorner.Parent =
    visualPanel
local visualPanelStroke =
    Instance.new("UIStroke")
visualPanelStroke.Color =
    Color3.fromRGB(
        205,
        120,
        255
    )
visualPanelStroke.Transparency =
    0.86
visualPanelStroke.Thickness =
    1
visualPanelStroke.Parent =
    visualPanel
local visualLabel =
    Instance.new("TextLabel")
visualLabel.Name =
    "VisualLabel"
visualLabel.Size =
    UDim2.new(
        1,
        -90,
        0,
        24
    )
visualLabel.Position =
    UDim2.fromOffset(
        14,
        10
    )
visualLabel.BackgroundTransparency =
    1
visualLabel.Text =
    "Visual Effect"
visualLabel.TextColor3 =
    Color3.fromRGB(
        242,
        225,
        250
    )
visualLabel.TextSize =
    14
visualLabel.Font =
    Enum.Font.GothamMedium
visualLabel.TextXAlignment =
    Enum.TextXAlignment.Left
visualLabel.TextYAlignment =
    Enum.TextYAlignment.Center
visualLabel.ZIndex =
    17
visualLabel.Parent =
    visualPanel
local visualDescription =
    Instance.new("TextLabel")
visualDescription.Name =
    "VisualDescription"
visualDescription.Size =
    UDim2.new(
        1,
        -70,
        0,
        20
    )
visualDescription.Position =
    UDim2.fromOffset(
        14,
        38
    )
visualDescription.BackgroundTransparency =
    1
visualDescription.Text =
    "Black cube follows your character"
visualDescription.TextColor3 =
    Color3.fromRGB(
        185,
        155,
        205
    )
visualDescription.TextSize =
    10
visualDescription.Font =
    Enum.Font.Gotham
visualDescription.TextXAlignment =
    Enum.TextXAlignment.Left
visualDescription.ZIndex =
    17
visualDescription.Parent =
    visualPanel
local visualCheckBox =
    Instance.new("TextButton")
visualCheckBox.Name =
    "VisualEffectCheckBox"
visualCheckBox.Size =
    UDim2.fromOffset(
        24,
        24
    )
visualCheckBox.Position =
    UDim2.new(
        1,
        -14,
        0.5,
        0
    )
visualCheckBox.AnchorPoint =
    Vector2.new(
        1,
        0.5
    )
visualCheckBox.BackgroundColor3 =
    Color3.fromRGB(
        68,
        24,
        96
    )
visualCheckBox.BackgroundTransparency =
    0
visualCheckBox.BorderSizePixel =
    0
visualCheckBox.Text =
    ""
visualCheckBox.AutoButtonColor =
    false
visualCheckBox.ZIndex =
    18
visualCheckBox.Parent =
    visualPanel
local visualCheckCorner =
    Instance.new("UICorner")
visualCheckCorner.CornerRadius =
    UDim.new(
        0,
        6
    )
visualCheckCorner.Parent =
    visualCheckBox
local visualCheckStroke =
    Instance.new("UIStroke")
visualCheckStroke.Color =
    Color3.fromRGB(
        180,
        105,
        225
    )
visualCheckStroke.Thickness =
    1
visualCheckStroke.Transparency =
    0.5
visualCheckStroke.Parent =
    visualCheckBox
local visualCheckMark =
    Instance.new("TextLabel")
visualCheckMark.Name =
    "Check"
visualCheckMark.Size =
    UDim2.fromScale(
        1,
        1
    )
visualCheckMark.Position =
    UDim2.fromScale(
        0,
        0
    )
visualCheckMark.BackgroundTransparency =
    1
visualCheckMark.Text =
    "✓"
visualCheckMark.TextColor3 =
    Color3.fromRGB(
        255,
        240,
        255
    )
visualCheckMark.TextSize =
    17
visualCheckMark.Font =
    Enum.Font.GothamMedium
visualCheckMark.TextTransparency =
    1
visualCheckMark.ZIndex =
    19
visualCheckMark.Parent =
    visualCheckBox
local targetHighlightColor = Color3.fromRGB(205, 120, 255)
local targetHighlightEnabled = false
local targetHighlightConnection = nil
local targetHighlight = nil
local targetHighlightCharacter = nil
local targetFillEnabled = false
local targetFillTransparency = 0.45
local targetOutlineTransparency = 0
local targetHighlightRow
local targetHighlightColorButton
local targetTransparencySliderDragging = false
local targetTransparencySliderActive
local targetOutlineSlider
local targetFillSlider
local targetColorPaletteOpenState = false
local targetColorPaletteVisibleState = false
local targetColorPaletteAnimationId = 0
local targetColorPaletteDragging = false
local targetColorPaletteDragStart
local targetColorPaletteStartPosition
local targetColorPalettePickingSV = false
local targetColorPalettePickingHue = false
local targetColorPalette = Instance.new("Frame")
targetColorPalette.Name = "TargetColorPalette"
targetColorPalette.Size = UDim2.fromOffset(246, 276)
targetColorPalette.BackgroundColor3 = Color3.fromRGB(72, 24, 100)
targetColorPalette.BackgroundTransparency = 0.02
targetColorPalette.BorderSizePixel = 0
targetColorPalette.Active = true
targetColorPalette.Visible = false
targetColorPalette.ZIndex = 50
targetColorPalette.Parent = gui
local targetColorPaletteCorner = Instance.new("UICorner")
targetColorPaletteCorner.CornerRadius = UDim.new(0, 14)
targetColorPaletteCorner.Parent = targetColorPalette
local targetColorPaletteStroke = Instance.new("UIStroke")
targetColorPaletteStroke.Color = Color3.fromRGB(205, 120, 255)
targetColorPaletteStroke.Transparency = 0.45
targetColorPaletteStroke.Thickness = 1
targetColorPaletteStroke.Parent = targetColorPalette
local targetColorPaletteTitleBar = Instance.new("Frame")
targetColorPaletteTitleBar.Name = "TitleBar"
targetColorPaletteTitleBar.Size = UDim2.new(1, -42, 0, 30)
targetColorPaletteTitleBar.Position = UDim2.fromOffset(6, 6)
targetColorPaletteTitleBar.BackgroundTransparency = 1
targetColorPaletteTitleBar.BorderSizePixel = 0
targetColorPaletteTitleBar.Active = true
targetColorPaletteTitleBar.ZIndex = 51
targetColorPaletteTitleBar.Parent = targetColorPalette
local targetColorPaletteTitle = Instance.new("TextLabel")
targetColorPaletteTitle.Name = "Title"
targetColorPaletteTitle.Size = UDim2.new(1, -38, 1, 0)
targetColorPaletteTitle.Position = UDim2.fromOffset(4, 0)
targetColorPaletteTitle.BackgroundTransparency = 1
targetColorPaletteTitle.Text = "Target Color"
targetColorPaletteTitle.TextColor3 = Color3.fromRGB(245, 225, 255)
targetColorPaletteTitle.TextSize = 14
targetColorPaletteTitle.Font = Enum.Font.GothamMedium
targetColorPaletteTitle.TextXAlignment = Enum.TextXAlignment.Left
targetColorPaletteTitle.TextYAlignment = Enum.TextYAlignment.Center
targetColorPaletteTitle.ZIndex = 52
targetColorPaletteTitle.Parent = targetColorPaletteTitleBar
local targetColorPaletteClose = Instance.new("TextButton")
targetColorPaletteClose.Name = "Close"
targetColorPaletteClose.Size = UDim2.fromOffset(26, 26)
targetColorPaletteClose.Position = UDim2.new(1, -8, 0, 21)
targetColorPaletteClose.AnchorPoint = Vector2.new(1, 0.5)
targetColorPaletteClose.BackgroundColor3 = Color3.fromRGB(94, 31, 122)
targetColorPaletteClose.BackgroundTransparency = 0.15
targetColorPaletteClose.BorderSizePixel = 0
targetColorPaletteClose.Text = "×"
targetColorPaletteClose.TextColor3 = Color3.fromRGB(245, 225, 255)
targetColorPaletteClose.TextSize = 18
targetColorPaletteClose.Font = Enum.Font.GothamMedium
targetColorPaletteClose.AutoButtonColor = false
targetColorPaletteClose.ZIndex = 52
targetColorPaletteClose.Parent = targetColorPalette
local targetColorPaletteCloseCorner = Instance.new("UICorner")
targetColorPaletteCloseCorner.CornerRadius = UDim.new(0, 7)
targetColorPaletteCloseCorner.Parent = targetColorPaletteClose
local targetColorSV = Instance.new("Frame")
targetColorSV.Name = "SaturationValue"
targetColorSV.Size = UDim2.fromOffset(188, 188)
targetColorSV.Position = UDim2.fromOffset(12, 46)
targetColorSV.BackgroundColor3 = targetHighlightColor
targetColorSV.BorderSizePixel = 0
targetColorSV.Active = true
targetColorSV.ClipsDescendants = true
targetColorSV.ZIndex = 51
targetColorSV.Parent = targetColorPalette
local targetColorSVCorner = Instance.new("UICorner")
targetColorSVCorner.CornerRadius = UDim.new(0, 10)
targetColorSVCorner.Parent = targetColorSV
local targetColorSVWhite = Instance.new("Frame")
targetColorSVWhite.Name = "WhiteLayer"
targetColorSVWhite.Size = UDim2.fromScale(1, 1)
targetColorSVWhite.BackgroundColor3 = Color3.new(1, 1, 1)
targetColorSVWhite.BorderSizePixel = 0
targetColorSVWhite.ZIndex = 52
targetColorSVWhite.Parent = targetColorSV
local targetColorSVWhiteGradient = Instance.new("UIGradient")
targetColorSVWhiteGradient.Color = ColorSequence.new(Color3.new(1, 1, 1), Color3.new(1, 1, 1))
targetColorSVWhiteGradient.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(1, 1)})
targetColorSVWhiteGradient.Rotation = 0
targetColorSVWhiteGradient.Parent = targetColorSVWhite
local targetColorSVBlack = Instance.new("Frame")
targetColorSVBlack.Name = "BlackLayer"
targetColorSVBlack.Size = UDim2.fromScale(1, 1)
targetColorSVBlack.BackgroundColor3 = Color3.new(0, 0, 0)
targetColorSVBlack.BorderSizePixel = 0
targetColorSVBlack.ZIndex = 53
targetColorSVBlack.Parent = targetColorSV
local targetColorSVBlackGradient = Instance.new("UIGradient")
targetColorSVBlackGradient.Color = ColorSequence.new(Color3.new(0, 0, 0), Color3.new(0, 0, 0))
targetColorSVBlackGradient.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(1, 0)})
targetColorSVBlackGradient.Rotation = 90
targetColorSVBlackGradient.Parent = targetColorSVBlack
local targetColorSVHitbox = Instance.new("TextButton")
targetColorSVHitbox.Name = "Input"
targetColorSVHitbox.Size = UDim2.fromScale(1, 1)
targetColorSVHitbox.BackgroundTransparency = 1
targetColorSVHitbox.BorderSizePixel = 0
targetColorSVHitbox.Text = ""
targetColorSVHitbox.AutoButtonColor = false
targetColorSVHitbox.ZIndex = 54
targetColorSVHitbox.Parent = targetColorSV
local targetColorSVMarker = Instance.new("Frame")
targetColorSVMarker.Name = "Marker"
targetColorSVMarker.Size = UDim2.fromOffset(12, 12)
targetColorSVMarker.AnchorPoint = Vector2.new(0.5, 0.5)
targetColorSVMarker.BackgroundColor3 = Color3.new(1, 1, 1)
targetColorSVMarker.BorderSizePixel = 0
targetColorSVMarker.ZIndex = 55
targetColorSVMarker.Parent = targetColorSV
local targetColorSVMarkerCorner = Instance.new("UICorner")
targetColorSVMarkerCorner.CornerRadius = UDim.new(1, 0)
targetColorSVMarkerCorner.Parent = targetColorSVMarker
local targetColorSVMarkerStroke = Instance.new("UIStroke")
targetColorSVMarkerStroke.Color = Color3.new(0, 0, 0)
targetColorSVMarkerStroke.Thickness = 2
targetColorSVMarkerStroke.Transparency = 0.2
targetColorSVMarkerStroke.Parent = targetColorSVMarker
local targetColorHue = Instance.new("Frame")
targetColorHue.Name = "Hue"
targetColorHue.Size = UDim2.fromOffset(18, 188)
targetColorHue.Position = UDim2.fromOffset(216, 46)
targetColorHue.BackgroundColor3 = Color3.new(1, 1, 1)
targetColorHue.BorderSizePixel = 0
targetColorHue.Active = true
targetColorHue.ClipsDescendants = true
targetColorHue.ZIndex = 51
targetColorHue.Parent = targetColorPalette
local targetColorHueCorner = Instance.new("UICorner")
targetColorHueCorner.CornerRadius = UDim.new(0, 9)
targetColorHueCorner.Parent = targetColorHue
local targetColorHueGradient = Instance.new("UIGradient")
targetColorHueGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
    ColorSequenceKeypoint.new(1 / 6, Color3.fromRGB(255, 255, 0)),
    ColorSequenceKeypoint.new(2 / 6, Color3.fromRGB(0, 255, 0)),
    ColorSequenceKeypoint.new(3 / 6, Color3.fromRGB(0, 255, 255)),
    ColorSequenceKeypoint.new(4 / 6, Color3.fromRGB(0, 0, 255)),
    ColorSequenceKeypoint.new(5 / 6, Color3.fromRGB(255, 0, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0))
})
targetColorHueGradient.Rotation = 90
targetColorHueGradient.Parent = targetColorHue
local targetColorHueHitbox = Instance.new("TextButton")
targetColorHueHitbox.Name = "Input"
targetColorHueHitbox.Size = UDim2.fromScale(1, 1)
targetColorHueHitbox.BackgroundTransparency = 1
targetColorHueHitbox.BorderSizePixel = 0
targetColorHueHitbox.Text = ""
targetColorHueHitbox.AutoButtonColor = false
targetColorHueHitbox.ZIndex = 54
targetColorHueHitbox.Parent = targetColorHue
local targetColorHueMarker = Instance.new("Frame")
targetColorHueMarker.Name = "Marker"
targetColorHueMarker.Size = UDim2.new(1, 4, 0, 5)
targetColorHueMarker.AnchorPoint = Vector2.new(0.5, 0.5)
targetColorHueMarker.BackgroundColor3 = Color3.new(1, 1, 1)
targetColorHueMarker.BorderSizePixel = 0
targetColorHueMarker.ZIndex = 55
targetColorHueMarker.Parent = targetColorHue
local targetColorHueMarkerCorner = Instance.new("UICorner")
targetColorHueMarkerCorner.CornerRadius = UDim.new(1, 0)
targetColorHueMarkerCorner.Parent = targetColorHueMarker
local targetColorPreview = Instance.new("Frame")
targetColorPreview.Name = "Preview"
targetColorPreview.Size = UDim2.fromOffset(38, 20)
targetColorPreview.Position = UDim2.fromOffset(12, 242)
targetColorPreview.BackgroundColor3 = targetHighlightColor
targetColorPreview.BorderSizePixel = 0
targetColorPreview.ZIndex = 51
targetColorPreview.Parent = targetColorPalette
local targetColorPreviewCorner = Instance.new("UICorner")
targetColorPreviewCorner.CornerRadius = UDim.new(0, 7)
targetColorPreviewCorner.Parent = targetColorPreview
local targetColorPreviewStroke = Instance.new("UIStroke")
targetColorPreviewStroke.Color = Color3.fromRGB(235, 210, 255)
targetColorPreviewStroke.Transparency = 0.35
targetColorPreviewStroke.Parent = targetColorPreview
local targetColorPreviewLabel = Instance.new("TextLabel")
targetColorPreviewLabel.Name = "Value"
targetColorPreviewLabel.Size = UDim2.new(1, -62, 0, 20)
targetColorPreviewLabel.Position = UDim2.fromOffset(58, 242)
targetColorPreviewLabel.BackgroundTransparency = 1
targetColorPreviewLabel.Text = "#CD78FF"
targetColorPreviewLabel.TextColor3 = Color3.fromRGB(225, 205, 235)
targetColorPreviewLabel.TextSize = 11
targetColorPreviewLabel.Font = Enum.Font.GothamMedium
targetColorPreviewLabel.TextXAlignment = Enum.TextXAlignment.Left
targetColorPreviewLabel.TextYAlignment = Enum.TextYAlignment.Center
targetColorPreviewLabel.ZIndex = 51
targetColorPreviewLabel.Parent = targetColorPalette
local function updateTargetHighlightVisuals()
    local h, s, v = targetHighlightColor:ToHSV()
    targetColorPreview.BackgroundColor3 = targetHighlightColor
    targetColorSV.BackgroundColor3 = Color3.fromHSV(h, 1, 1)
    targetColorSVMarker.Position = UDim2.new(s, 0, 1 - v, 0)
    targetColorHueMarker.Position = UDim2.new(0.5, 0, h, 0)
    targetHighlightColorButton.BackgroundColor3 = targetHighlightColor
    if targetOutlineSlider then
        targetOutlineSlider.fill.BackgroundColor3 = targetHighlightColor
    end
    if targetFillSlider then
        targetFillSlider.fill.BackgroundColor3 = targetHighlightColor
    end
    targetColorPreviewLabel.Text = string.format("#%02X%02X%02X", math.floor(targetHighlightColor.R * 255 + 0.5), math.floor(targetHighlightColor.G * 255 + 0.5), math.floor(targetHighlightColor.B * 255 + 0.5))
    if targetHighlight then
        targetHighlight.OutlineColor = targetHighlightColor
        targetHighlight.FillColor = targetHighlightColor
        targetHighlight.OutlineTransparency = targetOutlineTransparency
        targetHighlight.FillTransparency = targetFillEnabled and targetFillTransparency or 1
    end
end
local function setTargetHighlightColor(color)
    targetHighlightColor = color
    updateTargetHighlightVisuals()
end
local function updateTargetColorFromSV(position)
    local absolutePosition = targetColorSV.AbsolutePosition
    local absoluteSize = targetColorSV.AbsoluteSize
    local x = math.clamp(position.X - absolutePosition.X, 0, absoluteSize.X)
    local y = math.clamp(position.Y - absolutePosition.Y, 0, absoluteSize.Y)
    local h = select(1, targetHighlightColor:ToHSV())
    local s = absoluteSize.X > 0 and x / absoluteSize.X or 0
    local v = absoluteSize.Y > 0 and 1 - y / absoluteSize.Y or 0
    setTargetHighlightColor(Color3.fromHSV(h, s, v))
end
local function updateTargetColorFromHue(position)
    local absolutePosition = targetColorHue.AbsolutePosition
    local absoluteSize = targetColorHue.AbsoluteSize
    local y = math.clamp(position.Y - absolutePosition.Y, 0, absoluteSize.Y)
    local h = absoluteSize.Y > 0 and y / absoluteSize.Y or 0
    local _, s, v = targetHighlightColor:ToHSV()
    setTargetHighlightColor(Color3.fromHSV(h, s, v))
end
local function getTargetColorPalettePosition()
    local rowPosition = targetHighlightRow.AbsolutePosition
    local rowSize = targetHighlightRow.AbsoluteSize
    local paletteSize = Vector2.new(246, 276)
    local viewport = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1366, 768)
    local x = rowPosition.X + rowSize.X + 12
    if x + paletteSize.X > viewport.X - 8 then
        x = rowPosition.X - paletteSize.X - 12
    end
    local y = rowPosition.Y
    x = math.clamp(x, 8, math.max(8, viewport.X - paletteSize.X - 8))
    y = math.clamp(y, 8, math.max(8, viewport.Y - paletteSize.Y - 8))
    return Vector2.new(x, y)
end
local function openTargetColorPalette()
    if not targetHighlightEnabled then
        return
    end
    targetColorPaletteOpenState = true
    targetColorPaletteAnimationId += 1
    local animationId = targetColorPaletteAnimationId
    local destination = getTargetColorPalettePosition()
    local start = destination + Vector2.new(-18, 0)
    targetColorPalette.Position = UDim2.fromOffset(start.X, start.Y)
    targetColorPalette.Visible = true
    targetColorPaletteVisibleState = true
    updateTargetHighlightVisuals()
    TweenService:Create(targetColorPalette, TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.fromOffset(destination.X, destination.Y)}):Play()
end
local function closeTargetColorPalette(animated)
    targetColorPaletteOpenState = false
    targetColorPaletteAnimationId += 1
    local animationId = targetColorPaletteAnimationId
    if not targetColorPalette.Visible then
        targetColorPaletteVisibleState = false
        return
    end
    if not animated then
        targetColorPalette.Visible = false
        targetColorPaletteVisibleState = false
        return
    end
    local current = targetColorPalette.Position
    local destination = UDim2.fromOffset(current.X.Offset - 18, current.Y.Offset)
    TweenService:Create(targetColorPalette, TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = destination}):Play()
    task.delay(0.16, function()
        if animationId == targetColorPaletteAnimationId then
            targetColorPalette.Visible = false
            targetColorPaletteVisibleState = false
        end
    end)
end
local function hideTargetColorPaletteForMenu()
    targetColorPaletteAnimationId += 1
    targetColorPalette.Visible = false
    targetColorPaletteVisibleState = false
end
targetColorPaletteClose.MouseButton1Click:Connect(function()
    closeTargetColorPalette(true)
end)
targetColorSVHitbox.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        targetColorPalettePickingSV = true
        updateTargetColorFromSV(input.Position)
    end
end)
targetColorHueHitbox.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        targetColorPalettePickingHue = true
        updateTargetColorFromHue(input.Position)
    end
end)
targetColorPaletteTitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        targetColorPaletteDragging = true
        targetColorPaletteDragStart = input.Position
        targetColorPaletteStartPosition = Vector2.new(targetColorPalette.Position.X.Offset, targetColorPalette.Position.Y.Offset)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType ~= Enum.UserInputType.MouseMovement then
        return
    end
    if targetColorPalettePickingSV then
        updateTargetColorFromSV(input.Position)
    end
    if targetColorPalettePickingHue then
        updateTargetColorFromHue(input.Position)
    end
    if targetColorPaletteDragging then
        local delta = input.Position - targetColorPaletteDragStart
        local viewport = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1366, 768)
        local x = math.clamp(targetColorPaletteStartPosition.X + delta.X, 8, math.max(8, viewport.X - 246 - 8))
        local y = math.clamp(targetColorPaletteStartPosition.Y + delta.Y, 8, math.max(8, viewport.Y - 276 - 8))
        targetColorPalette.Position = UDim2.fromOffset(x, y)
    end
    if targetTransparencySliderDragging and targetTransparencySliderActive then
        if targetTransparencySliderActive == targetOutlineSlider.hitbox then
            local track = targetOutlineSlider.track
            local t = math.clamp((input.Position.X - track.AbsolutePosition.X) / math.max(track.AbsoluteSize.X, 1), 0, 1)
            targetOutlineSlider.update(1 - t)
        elseif targetTransparencySliderActive == targetFillSlider.hitbox then
            local track = targetFillSlider.track
            local t = math.clamp((input.Position.X - track.AbsolutePosition.X) / math.max(track.AbsoluteSize.X, 1), 0, 1)
            targetFillSlider.update(1 - t)
        end
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        targetColorPalettePickingSV = false
        targetColorPalettePickingHue = false
        targetColorPaletteDragging = false
        targetTransparencySliderDragging = false
        targetTransparencySliderActive = nil
    end
end)
targetHighlightRow = Instance.new("Frame")
targetHighlightRow.Name = "TargetHighlightRow"
targetHighlightRow.Size = UDim2.new(1, -20, 0, 76)
targetHighlightRow.Position = UDim2.fromOffset(10, 174)
targetHighlightRow.BackgroundColor3 = Color3.fromRGB(104, 31, 151)
targetHighlightRow.BackgroundTransparency = 0.3
targetHighlightRow.BorderSizePixel = 0
targetHighlightRow.ZIndex = 16
targetHighlightRow.Visible = false
targetHighlightRow.Parent = content
local targetHighlightRowCorner = Instance.new("UICorner")
targetHighlightRowCorner.CornerRadius = UDim.new(0, 12)
targetHighlightRowCorner.Parent = targetHighlightRow
local targetHighlightRowStroke = Instance.new("UIStroke")
targetHighlightRowStroke.Color = Color3.fromRGB(205, 120, 255)
targetHighlightRowStroke.Transparency = 0.86
targetHighlightRowStroke.Thickness = 1
targetHighlightRowStroke.Parent = targetHighlightRow
local targetHighlightLabel = Instance.new("TextLabel")
targetHighlightLabel.Name = "TargetHighlightLabel"
targetHighlightLabel.Size = UDim2.new(1, -120, 0, 24)
targetHighlightLabel.Position = UDim2.fromOffset(14, 10)
targetHighlightLabel.BackgroundTransparency = 1
targetHighlightLabel.Text = "Target Highlight"
targetHighlightLabel.TextColor3 = Color3.fromRGB(242, 225, 250)
targetHighlightLabel.TextSize = 14
targetHighlightLabel.Font = Enum.Font.GothamMedium
targetHighlightLabel.TextXAlignment = Enum.TextXAlignment.Left
targetHighlightLabel.TextYAlignment = Enum.TextYAlignment.Center
targetHighlightLabel.ZIndex = 17
targetHighlightLabel.Parent = targetHighlightRow
local targetHighlightDescription = Instance.new("TextLabel")
targetHighlightDescription.Name = "TargetHighlightDescription"
targetHighlightDescription.Size = UDim2.new(1, -120, 0, 20)
targetHighlightDescription.Position = UDim2.fromOffset(14, 38)
targetHighlightDescription.BackgroundTransparency = 1
targetHighlightDescription.Text = "Highlights the player under your cursor"
targetHighlightDescription.TextColor3 = Color3.fromRGB(185, 155, 205)
targetHighlightDescription.TextSize = 10
targetHighlightDescription.Font = Enum.Font.Gotham
targetHighlightDescription.TextXAlignment = Enum.TextXAlignment.Left
targetHighlightDescription.ZIndex = 17
targetHighlightDescription.Parent = targetHighlightRow
targetHighlightColorButton = Instance.new("TextButton")
targetHighlightColorButton.Name = "ColorButton"
targetHighlightColorButton.Size = UDim2.fromOffset(28, 24)
targetHighlightColorButton.Position = UDim2.new(1, -48, 0.5, 0)
targetHighlightColorButton.AnchorPoint = Vector2.new(1, 0.5)
targetHighlightColorButton.BackgroundColor3 = targetHighlightColor
targetHighlightColorButton.BackgroundTransparency = 0
targetHighlightColorButton.BorderSizePixel = 0
targetHighlightColorButton.Text = ""
targetHighlightColorButton.AutoButtonColor = false
targetHighlightColorButton.ZIndex = 18
targetHighlightColorButton.Parent = targetHighlightRow
local targetHighlightColorButtonCorner = Instance.new("UICorner")
targetHighlightColorButtonCorner.CornerRadius = UDim.new(0, 7)
targetHighlightColorButtonCorner.Parent = targetHighlightColorButton
local targetHighlightColorButtonStroke = Instance.new("UIStroke")
targetHighlightColorButtonStroke.Color = Color3.fromRGB(238, 220, 250)
targetHighlightColorButtonStroke.Transparency = 0.35
targetHighlightColorButtonStroke.Thickness = 1
targetHighlightColorButtonStroke.Parent = targetHighlightColorButton
local targetHighlightCheckBox = Instance.new("TextButton")
targetHighlightCheckBox.Name = "TargetHighlightCheckBox"
targetHighlightCheckBox.Size = UDim2.fromOffset(24, 24)
targetHighlightCheckBox.Position = UDim2.new(1, -14, 0.5, 0)
targetHighlightCheckBox.AnchorPoint = Vector2.new(1, 0.5)
targetHighlightCheckBox.BackgroundColor3 = Color3.fromRGB(68, 24, 96)
targetHighlightCheckBox.BackgroundTransparency = 0
targetHighlightCheckBox.BorderSizePixel = 0
targetHighlightCheckBox.Text = ""
targetHighlightCheckBox.AutoButtonColor = false
targetHighlightCheckBox.ZIndex = 18
targetHighlightCheckBox.Parent = targetHighlightRow
local targetHighlightCheckCorner = Instance.new("UICorner")
targetHighlightCheckCorner.CornerRadius = UDim.new(0, 6)
targetHighlightCheckCorner.Parent = targetHighlightCheckBox
local targetHighlightCheckStroke = Instance.new("UIStroke")
targetHighlightCheckStroke.Color = Color3.fromRGB(180, 105, 225)
targetHighlightCheckStroke.Thickness = 1
targetHighlightCheckStroke.Transparency = 0.5
targetHighlightCheckStroke.Parent = targetHighlightCheckBox
local targetHighlightCheckMark = Instance.new("TextLabel")
targetHighlightCheckMark.Name = "Check"
targetHighlightCheckMark.Size = UDim2.fromScale(1, 1)
targetHighlightCheckMark.Position = UDim2.fromScale(0, 0)
targetHighlightCheckMark.BackgroundTransparency = 1
targetHighlightCheckMark.Text = "✓"
targetHighlightCheckMark.TextColor3 = Color3.fromRGB(255, 240, 255)
targetHighlightCheckMark.TextSize = 17
targetHighlightCheckMark.Font = Enum.Font.GothamMedium
targetHighlightCheckMark.TextTransparency = 1
targetHighlightCheckMark.ZIndex = 19
targetHighlightCheckMark.Parent = targetHighlightCheckBox
local targetHighlightFillRow = Instance.new("Frame")
targetHighlightFillRow.Name = "TargetHighlightFillRow"
targetHighlightFillRow.Size = UDim2.new(1, -20, 0, 76)
targetHighlightFillRow.Position = UDim2.fromOffset(10, 258)
targetHighlightFillRow.BackgroundColor3 = Color3.fromRGB(104, 31, 151)
targetHighlightFillRow.BackgroundTransparency = 0.3
targetHighlightFillRow.BorderSizePixel = 0
targetHighlightFillRow.ZIndex = 16
targetHighlightFillRow.Visible = false
targetHighlightFillRow.Parent = content
local targetHighlightFillRowCorner = Instance.new("UICorner")
targetHighlightFillRowCorner.CornerRadius = UDim.new(0, 12)
targetHighlightFillRowCorner.Parent = targetHighlightFillRow
local targetHighlightFillRowStroke = Instance.new("UIStroke")
targetHighlightFillRowStroke.Color = Color3.fromRGB(205, 120, 255)
targetHighlightFillRowStroke.Transparency = 0.86
targetHighlightFillRowStroke.Thickness = 1
targetHighlightFillRowStroke.Parent = targetHighlightFillRow
local targetHighlightFillLabel = Instance.new("TextLabel")
targetHighlightFillLabel.Name = "TargetHighlightFillLabel"
targetHighlightFillLabel.Size = UDim2.new(1, -90, 0, 24)
targetHighlightFillLabel.Position = UDim2.fromOffset(14, 10)
targetHighlightFillLabel.BackgroundTransparency = 1
targetHighlightFillLabel.Text = "Target Fill"
targetHighlightFillLabel.TextColor3 = Color3.fromRGB(242, 225, 250)
targetHighlightFillLabel.TextSize = 14
targetHighlightFillLabel.Font = Enum.Font.GothamMedium
targetHighlightFillLabel.TextXAlignment = Enum.TextXAlignment.Left
targetHighlightFillLabel.TextYAlignment = Enum.TextYAlignment.Center
targetHighlightFillLabel.ZIndex = 17
targetHighlightFillLabel.Parent = targetHighlightFillRow
local targetHighlightFillDescription = Instance.new("TextLabel")
targetHighlightFillDescription.Name = "TargetHighlightFillDescription"
targetHighlightFillDescription.Size = UDim2.new(1, -70, 0, 20)
targetHighlightFillDescription.Position = UDim2.fromOffset(14, 38)
targetHighlightFillDescription.BackgroundTransparency = 1
targetHighlightFillDescription.Text = "Fills the selected player with the target color"
targetHighlightFillDescription.TextColor3 = Color3.fromRGB(185, 155, 205)
targetHighlightFillDescription.TextSize = 10
targetHighlightFillDescription.Font = Enum.Font.Gotham
targetHighlightFillDescription.TextXAlignment = Enum.TextXAlignment.Left
targetHighlightFillDescription.ZIndex = 17
targetHighlightFillDescription.Parent = targetHighlightFillRow
local targetHighlightFillCheckBox = Instance.new("TextButton")
targetHighlightFillCheckBox.Name = "TargetHighlightFillCheckBox"
targetHighlightFillCheckBox.Size = UDim2.fromOffset(24, 24)
targetHighlightFillCheckBox.Position = UDim2.new(1, -14, 0.5, 0)
targetHighlightFillCheckBox.AnchorPoint = Vector2.new(1, 0.5)
targetHighlightFillCheckBox.BackgroundColor3 = Color3.fromRGB(68, 24, 96)
targetHighlightFillCheckBox.BackgroundTransparency = 0
targetHighlightFillCheckBox.BorderSizePixel = 0
targetHighlightFillCheckBox.Text = ""
targetHighlightFillCheckBox.AutoButtonColor = false
targetHighlightFillCheckBox.ZIndex = 18
targetHighlightFillCheckBox.Parent = targetHighlightFillRow
local targetHighlightFillCheckCorner = Instance.new("UICorner")
targetHighlightFillCheckCorner.CornerRadius = UDim.new(0, 6)
targetHighlightFillCheckCorner.Parent = targetHighlightFillCheckBox
local targetHighlightFillCheckStroke = Instance.new("UIStroke")
targetHighlightFillCheckStroke.Color = Color3.fromRGB(180, 105, 225)
targetHighlightFillCheckStroke.Thickness = 1
targetHighlightFillCheckStroke.Transparency = 0.5
targetHighlightFillCheckStroke.Parent = targetHighlightFillCheckBox
local targetHighlightFillCheckMark = Instance.new("TextLabel")
targetHighlightFillCheckMark.Name = "Check"
targetHighlightFillCheckMark.Size = UDim2.fromScale(1, 1)
targetHighlightFillCheckMark.BackgroundTransparency = 1
targetHighlightFillCheckMark.Text = "✓"
targetHighlightFillCheckMark.TextColor3 = Color3.fromRGB(255, 240, 255)
targetHighlightFillCheckMark.TextSize = 17
targetHighlightFillCheckMark.Font = Enum.Font.GothamMedium
targetHighlightFillCheckMark.TextTransparency = 1
targetHighlightFillCheckMark.ZIndex = 19
targetHighlightFillCheckMark.Parent = targetHighlightFillCheckBox
local function createTransparencySlider(parent, name, labelText, initialValue, valueGetter, valueSetter, rightInset)
    local label = Instance.new("TextLabel")
    label.Name = name .. "Label"
    label.Size = UDim2.fromOffset(78, 14)
    label.Position = UDim2.fromOffset(14, 57)
    label.BackgroundTransparency = 1
    label.Text = labelText
    label.TextColor3 = Color3.fromRGB(185, 155, 205)
    label.TextSize = 9
    label.Font = Enum.Font.GothamMedium
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextYAlignment = Enum.TextYAlignment.Center
    label.ZIndex = 17
    label.Parent = parent
    local valueLabel = Instance.new("TextLabel")
    valueLabel.Name = name .. "Value"
    valueLabel.Size = UDim2.fromOffset(38, 14)
    valueLabel.Position = UDim2.fromOffset(92, 57)
    valueLabel.BackgroundTransparency = 1
    valueLabel.TextColor3 = Color3.fromRGB(225, 205, 235)
    valueLabel.TextSize = 9
    valueLabel.Font = Enum.Font.GothamMedium
    valueLabel.TextXAlignment = Enum.TextXAlignment.Left
    valueLabel.TextYAlignment = Enum.TextYAlignment.Center
    valueLabel.ZIndex = 17
    valueLabel.Parent = parent
    local track = Instance.new("Frame")
    track.Name = name .. "Track"
    track.Size = UDim2.new(1, -rightInset, 0, 6)
    track.Position = UDim2.fromOffset(136, 61)
    track.BackgroundColor3 = Color3.fromRGB(65, 23, 91)
    track.BackgroundTransparency = 0
    track.BorderSizePixel = 0
    track.Active = true
    track.ZIndex = 18
    track.Parent = parent
    local trackCorner = Instance.new("UICorner")
    trackCorner.CornerRadius = UDim.new(1, 0)
    trackCorner.Parent = track
    local fill = Instance.new("Frame")
    fill.Name = "Fill"
    fill.Size = UDim2.fromScale(1 - initialValue, 1)
    fill.BackgroundColor3 = targetHighlightColor
    fill.BorderSizePixel = 0
    fill.ZIndex = 19
    fill.Parent = track
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1, 0)
    fillCorner.Parent = fill
    local knob = Instance.new("Frame")
    knob.Name = "Knob"
    knob.Size = UDim2.fromOffset(10, 10)
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.BackgroundColor3 = Color3.fromRGB(245, 225, 255)
    knob.BorderSizePixel = 0
    knob.ZIndex = 20
    knob.Parent = track
    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius = UDim.new(1, 0)
    knobCorner.Parent = knob
    local knobStroke = Instance.new("UIStroke")
    knobStroke.Color = Color3.fromRGB(205, 120, 255)
    knobStroke.Thickness = 1
    knobStroke.Transparency = 0.25
    knobStroke.Parent = knob
    local hitbox = Instance.new("TextButton")
    hitbox.Name = name .. "Input"
    hitbox.Size = UDim2.new(1, 0, 0, 22)
    hitbox.Position = UDim2.new(0, 0, 0.5, 0)
    hitbox.AnchorPoint = Vector2.new(0, 0.5)
    hitbox.BackgroundTransparency = 1
    hitbox.BorderSizePixel = 0
    hitbox.Text = ""
    hitbox.AutoButtonColor = false
    hitbox.Active = true
    hitbox.ZIndex = 21
    hitbox.Parent = track
    local function update(value)
        value = math.clamp(value, 0, 1)
        valueSetter(value)
        fill.Size = UDim2.fromScale(1 - value, 1)
        knob.Position = UDim2.new(1 - value, 0, 0.5, 0)
        valueLabel.Text = string.format("%d%%", math.floor(value * 100 + 0.5))
        fill.BackgroundColor3 = targetHighlightColor
    end
    local function updateFromPosition(position)
        local absolutePosition = track.AbsolutePosition
        local absoluteSize = track.AbsoluteSize
        if absoluteSize.X <= 0 then
            return
        end
        local t = math.clamp((position.X - absolutePosition.X) / absoluteSize.X, 0, 1)
        update(1 - t)
    end
    hitbox.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            targetTransparencySliderDragging = true
            targetTransparencySliderActive = hitbox
            updateFromPosition(input.Position)
        end
    end)
    local slider = {
        label = label,
        valueLabel = valueLabel,
        track = track,
        fill = fill,
        knob = knob,
        hitbox = hitbox,
        update = function(value)
            update(value)
        end,
        setVisible = function(visible)
            label.Visible = visible
            valueLabel.Visible = visible
            track.Visible = visible
        end,
        isActive = function()
            return targetTransparencySliderActive == hitbox
        end,
        getValue = valueGetter
    }
    update(initialValue)
    return slider
end
targetOutlineSlider = createTransparencySlider(targetHighlightRow, "TargetOutlineTransparency", "Outline", targetOutlineTransparency, function()
    return targetOutlineTransparency
end, function(value)
    targetOutlineTransparency = value
    if targetHighlight then
        targetHighlight.OutlineTransparency = value
    end
end, 118)
targetFillSlider = createTransparencySlider(targetHighlightFillRow, "TargetFillTransparency", "Fill", targetFillTransparency, function()
    return targetFillTransparency
end, function(value)
    targetFillTransparency = value
    if targetHighlight and targetFillEnabled then
        targetHighlight.FillTransparency = value
    end
end, 78)
local function destroyTargetHighlight()
    if targetHighlightConnection then
        targetHighlightConnection:Disconnect()
        targetHighlightConnection = nil
    end
    if targetHighlight then
        targetHighlight:Destroy()
        targetHighlight = nil
    end
    targetHighlightCharacter = nil
end
local function updateTargetHighlight()
    if not targetHighlightEnabled or not targetHighlight then
        return
    end
    local currentCamera = workspace.CurrentCamera
    if not currentCamera then
        return
    end
    local mousePosition = UserInputService:GetMouseLocation()
    local ray = currentCamera:ViewportPointToRay(
        mousePosition.X,
        mousePosition.Y
    )
    local character = player.Character
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    raycastParams.FilterDescendantsInstances = character and {character} or {}
    raycastParams.IgnoreWater = true
    local result = workspace:Raycast(
        ray.Origin,
        ray.Direction * 1000,
        raycastParams
    )
    local targetCharacter = nil
    if result then
        local model = result.Instance:FindFirstAncestorOfClass("Model")
        if model then
            local humanoid = model:FindFirstChildOfClass("Humanoid")
            local targetPlayer = Players:GetPlayerFromCharacter(model)
            if humanoid and targetPlayer and targetPlayer ~= player then
                targetCharacter = model
            end
        end
    end
    if targetCharacter ~= targetHighlightCharacter then
        targetHighlightCharacter = targetCharacter
        targetHighlight.Adornee = targetCharacter
        targetHighlight.Enabled = targetCharacter ~= nil
    end
end
local function setTargetHighlightEnabled(enabled)
    targetHighlightEnabled = enabled
    TweenService:Create(
        targetHighlightCheckBox,
        TweenInfo.new(0.16, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
        {
            BackgroundColor3 = enabled and Color3.fromRGB(158, 72, 210) or Color3.fromRGB(68, 24, 96)
        }
    ):Play()
    TweenService:Create(
        targetHighlightCheckStroke,
        TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {
            Transparency = enabled and 0.2 or 0.5
        }
    ):Play()
    TweenService:Create(
        targetHighlightCheckMark,
        TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {
            TextTransparency = enabled and 0 or 1
        }
    ):Play()
    TweenService:Create(
        targetHighlightRowStroke,
        TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {
            Transparency = enabled and 0.68 or 0.86
        }
    ):Play()
    targetOutlineSlider.setVisible(enabled)
    targetFillSlider.setVisible(enabled)
    if enabled then
        targetHighlightFillRow.Visible = true
        destroyTargetHighlight()
        targetHighlightEnabled = true
        local newHighlight = Instance.new("Highlight")
        newHighlight.Name = "TargetHighlight"
        newHighlight.FillTransparency = targetFillEnabled and targetFillTransparency or 1
        newHighlight.OutlineTransparency = targetOutlineTransparency
        newHighlight.OutlineColor = targetHighlightColor
        newHighlight.FillColor = targetHighlightColor
        newHighlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        newHighlight.Enabled = false
        newHighlight.Parent = player.Character or workspace
        targetHighlight = newHighlight
        updateTargetHighlightVisuals()
        targetHighlightConnection = RunService.RenderStepped:Connect(updateTargetHighlight)
        openTargetColorPalette()
    else
        destroyTargetHighlight()
        targetHighlightFillRow.Visible = false
        closeTargetColorPalette(true)
    end
end
targetHighlightCheckBox.MouseButton1Click:Connect(function()
    setTargetHighlightEnabled(not targetHighlightEnabled)
end)
targetHighlightColorButton.MouseButton1Click:Connect(function()
    if not targetHighlightEnabled then
        return
    end
    if targetColorPaletteOpenState then
        closeTargetColorPalette(true)
    else
        openTargetColorPalette()
    end
end)
local function setTargetFillEnabled(enabled)
    targetFillEnabled = enabled
    TweenService:Create(targetHighlightFillCheckBox, TweenInfo.new(0.16, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {BackgroundColor3 = enabled and Color3.fromRGB(158, 72, 210) or Color3.fromRGB(68, 24, 96)}):Play()
    TweenService:Create(targetHighlightFillCheckStroke, TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Transparency = enabled and 0.2 or 0.5}):Play()
    TweenService:Create(targetHighlightFillCheckMark, TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {TextTransparency = enabled and 0 or 1}):Play()
    TweenService:Create(targetHighlightFillRowStroke, TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Transparency = enabled and 0.68 or 0.86}):Play()
    if targetHighlight then
        targetHighlight.FillColor = targetHighlightColor
        targetHighlight.FillTransparency = enabled and targetFillTransparency or 1
    end
end
targetHighlightFillCheckBox.MouseButton1Click:Connect(function()
    setTargetFillEnabled(not targetFillEnabled)
end)
targetOutlineSlider.update(targetOutlineTransparency)
targetFillSlider.update(targetFillTransparency)
targetOutlineSlider.setVisible(false)
targetFillSlider.setVisible(false)
updateTargetHighlightVisuals()
local aimbotPanel = Instance.new("Frame")
aimbotPanel.Name = "AimbotPanel"
aimbotPanel.Size = UDim2.new(1, -20, 0, 76)
aimbotPanel.Position = UDim2.fromOffset(10, 90)
aimbotPanel.BackgroundColor3 = Color3.fromRGB(104, 31, 151)
aimbotPanel.BackgroundTransparency = 0.3
aimbotPanel.BorderSizePixel = 0
aimbotPanel.Visible = false
aimbotPanel.ZIndex = 16
aimbotPanel.Parent = content
local aimbotPanelCorner = Instance.new("UICorner")
aimbotPanelCorner.CornerRadius = UDim.new(0, 12)
aimbotPanelCorner.Parent = aimbotPanel
local aimbotPanelStroke = Instance.new("UIStroke")
aimbotPanelStroke.Color = Color3.fromRGB(205, 120, 255)
aimbotPanelStroke.Transparency = 0.86
aimbotPanelStroke.Thickness = 1
aimbotPanelStroke.Parent = aimbotPanel
local aimbotLabel = Instance.new("TextLabel")
aimbotLabel.Name = "AimbotMouseLabel"
aimbotLabel.Size = UDim2.new(1, -90, 0, 24)
aimbotLabel.Position = UDim2.fromOffset(14, 10)
aimbotLabel.BackgroundTransparency = 1
aimbotLabel.Text = "Aimbot Mouse"
aimbotLabel.TextColor3 = Color3.fromRGB(242, 225, 250)
aimbotLabel.TextSize = 14
aimbotLabel.Font = Enum.Font.GothamMedium
aimbotLabel.TextXAlignment = Enum.TextXAlignment.Left
aimbotLabel.TextYAlignment = Enum.TextYAlignment.Center
aimbotLabel.ZIndex = 17
aimbotLabel.Parent = aimbotPanel
local aimbotDescription = Instance.new("TextLabel")
aimbotDescription.Name = "AimbotMouseDescription"
aimbotDescription.Size = UDim2.new(1, -70, 0, 20)
aimbotDescription.Position = UDim2.fromOffset(14, 38)
aimbotDescription.BackgroundTransparency = 1
aimbotDescription.Text = "Character faces the mouse direction"
aimbotDescription.TextColor3 = Color3.fromRGB(185, 155, 205)
aimbotDescription.TextSize = 10
aimbotDescription.Font = Enum.Font.Gotham
aimbotDescription.TextXAlignment = Enum.TextXAlignment.Left
aimbotDescription.ZIndex = 17
aimbotDescription.Parent = aimbotPanel
local aimbotCheckBox = Instance.new("TextButton")
aimbotCheckBox.Name = "AimbotMouseCheckBox"
aimbotCheckBox.Size = UDim2.fromOffset(24, 24)
aimbotCheckBox.Position = UDim2.new(1, -14, 0.5, 0)
aimbotCheckBox.AnchorPoint = Vector2.new(1, 0.5)
aimbotCheckBox.BackgroundColor3 = Color3.fromRGB(68, 24, 96)
aimbotCheckBox.BackgroundTransparency = 0
aimbotCheckBox.BorderSizePixel = 0
aimbotCheckBox.Text = ""
aimbotCheckBox.AutoButtonColor = false
aimbotCheckBox.ZIndex = 18
aimbotCheckBox.Parent = aimbotPanel
local aimbotCheckCorner = Instance.new("UICorner")
aimbotCheckCorner.CornerRadius = UDim.new(0, 6)
aimbotCheckCorner.Parent = aimbotCheckBox
local aimbotCheckStroke = Instance.new("UIStroke")
aimbotCheckStroke.Color = Color3.fromRGB(180, 105, 225)
aimbotCheckStroke.Thickness = 1
aimbotCheckStroke.Transparency = 0.5
aimbotCheckStroke.Parent = aimbotCheckBox
local aimbotCheckMark = Instance.new("TextLabel")
aimbotCheckMark.Name = "Check"
aimbotCheckMark.Size = UDim2.fromScale(1, 1)
aimbotCheckMark.Position = UDim2.fromScale(0, 0)
aimbotCheckMark.BackgroundTransparency = 1
aimbotCheckMark.Text = "✓"
aimbotCheckMark.TextColor3 = Color3.fromRGB(255, 240, 255)
aimbotCheckMark.TextSize = 17
aimbotCheckMark.Font = Enum.Font.GothamMedium
aimbotCheckMark.TextTransparency = 1
aimbotCheckMark.ZIndex = 19
aimbotCheckMark.Parent = aimbotCheckBox
local aimbotMouseEnabled = false
local aimbotMouseConnection = nil
local aimbotHumanoid = nil
local function setAimbotMouseEnabled(enabled)
    aimbotMouseEnabled = enabled
    TweenService:Create(
        aimbotCheckBox,
        TweenInfo.new(0.16, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
        {
            BackgroundColor3 = enabled and Color3.fromRGB(158, 72, 210) or Color3.fromRGB(68, 24, 96)
        }
    ):Play()
    TweenService:Create(
        aimbotCheckStroke,
        TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {
            Transparency = enabled and 0.2 or 0.5
        }
    ):Play()
    TweenService:Create(
        aimbotCheckMark,
        TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {
            TextTransparency = enabled and 0 or 1
        }
    ):Play()
    if enabled then
        if aimbotMouseConnection then
            aimbotMouseConnection:Disconnect()
        end
        local character = player.Character
        aimbotHumanoid = character and character:FindFirstChildOfClass("Humanoid") or nil
        if aimbotHumanoid then
            aimbotHumanoid.AutoRotate = false
        end
        aimbotMouseConnection = RunService.RenderStepped:Connect(function()
            if not aimbotMouseEnabled then
                return
            end
            local character = player.Character
            if not character then
                return
            end
            local root = character:FindFirstChild("HumanoidRootPart")
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if not root or not humanoid then
                return
            end
            aimbotHumanoid = humanoid
            humanoid.AutoRotate = false
            local currentCamera = workspace.CurrentCamera
            if not currentCamera then
                return
            end
            local mousePosition = UserInputService:GetMouseLocation()
            local ray = currentCamera:ViewportPointToRay(
                mousePosition.X,
                mousePosition.Y
            )
            local direction = Vector3.new(
                ray.Direction.X,
                0,
                ray.Direction.Z
            )
            if direction.Magnitude < 0.001 then
                return
            end
            direction = direction.Unit
            local targetCFrame =
                CFrame.lookAt(
                    root.Position,
                    root.Position + direction
                )
            root.CFrame =
                root.CFrame:Lerp(
                    targetCFrame,
                    math.clamp(
                        0.35,
                        0,
                        1
                    )
                )
        end)
    else
        if aimbotMouseConnection then
            aimbotMouseConnection:Disconnect()
            aimbotMouseConnection = nil
        end
        if aimbotHumanoid and aimbotHumanoid.Parent then
            aimbotHumanoid.AutoRotate = true
        end
        aimbotHumanoid = nil
    end
end
aimbotCheckBox.MouseButton1Click:Connect(function()
    setAimbotMouseEnabled(
        not aimbotMouseEnabled
    )
end)
local visualEnabled =
    false
local visualCube =
    nil
local visualCubeConnection =
    nil
local visualCubePosition =
    nil
local function destroyVisualCube()
    if visualCubeConnection then
        visualCubeConnection:Disconnect()
        visualCubeConnection =
            nil
    end
    if visualCube then
        visualCube:Destroy()
        visualCube =
            nil
    end
    visualCubePosition =
        nil
end
local function createVisualCube()
    destroyVisualCube()
    local cube =
        Instance.new("Part")
    cube.Name =
        "VisualEffectCube"
    cube.Shape =
        Enum.PartType.Block
    cube.Size =
        Vector3.new(
            4,
            4,
            4
        )
    cube.Color =
        Color3.fromRGB(
            0,
            0,
            0
        )
    cube.Material =
        Enum.Material.SmoothPlastic
    cube.Transparency =
        0
    cube.Anchored =
        true
    cube.CanCollide =
        false
    cube.CanTouch =
        false
    cube.CanQuery =
        false
    cube.CastShadow =
        false
    cube.Massless =
        true
    cube.Parent =
        workspace
    visualCube =
        cube
    visualCubeConnection =
        RunService.Heartbeat:Connect(
            function(dt)
                if not visualEnabled then
                    return
                end
                if not visualCube or
                    not visualCube.Parent then
                    return
                end
                local character =
                    player.Character
                if not character then
                    return
                end
                local root =
                    character:FindFirstChild(
                        "HumanoidRootPart"
                    )
                if not root then
                    return
                end
                local targetPosition =
                    root.Position -
                    root.CFrame.LookVector *
                    5
                targetPosition =
                    targetPosition +
                    Vector3.new(
                        0,
                        2,
                        0
                    )
                if not visualCubePosition then
                    visualCubePosition =
                        targetPosition
                end
                local alpha =
                    1 -
                    math.pow(
                        0.001,
                        dt
                    )
                visualCubePosition =
                    visualCubePosition:Lerp(
                        targetPosition,
                        alpha
                    )
                local targetCFrame =
                    CFrame.new(
                        visualCubePosition
                    )
                visualCube.CFrame =
                    visualCube.CFrame:Lerp(
                        targetCFrame,
                        math.clamp(
                            dt * 10,
                            0,
                            1
                        )
                    )
            end
        )
end
local function setVisualEnabled(enabled)
    visualEnabled =
        enabled
    TweenService:Create(
        visualCheckBox,
        TweenInfo.new(
            0.16,
            Enum.EasingStyle.Quart,
            Enum.EasingDirection.Out
        ),
        {
            BackgroundColor3 =
                enabled
                and Color3.fromRGB(
                    158,
                    72,
                    210
                )
                or Color3.fromRGB(
                    68,
                    24,
                    96
                )
        }
    ):Play()
    TweenService:Create(
        visualCheckStroke,
        TweenInfo.new(
            0.16,
            Enum.EasingStyle.Quad,
            Enum.EasingDirection.Out
        ),
        {
            Transparency =
                enabled
                and 0.2
                or 0.5
        }
    ):Play()
    TweenService:Create(
        visualCheckMark,
        TweenInfo.new(
            0.14,
            Enum.EasingStyle.Quad,
            Enum.EasingDirection.Out
        ),
        {
            TextTransparency =
                enabled
                and 0
                or 1
        }
    ):Play()
    TweenService:Create(
        visualPanelStroke,
        TweenInfo.new(
            0.16,
            Enum.EasingStyle.Quad,
            Enum.EasingDirection.Out
        ),
        {
            Transparency =
                enabled
                and 0.68
                or 0.86
        }
    ):Play()
    if enabled then
        createVisualCube()
    else
        destroyVisualCube()
    end
end
visualCheckBox.MouseButton1Click:Connect(function()
    setVisualEnabled(
        not visualEnabled
    )
end)
local function updateContentForCategory(name)
    visualPanel.Visible = name == "Visual"
    targetHighlightRow.Visible = name == "Visual"
    targetHighlightFillRow.Visible = name == "Visual" and targetHighlightEnabled
    aimbotPanel.Visible = name == "Misc"
    if name ~= "Visual" then
        closeTargetColorPalette(false)
    elseif targetHighlightEnabled and targetColorPaletteOpenState then
        openTargetColorPalette()
    end
end
local camera = workspace.CurrentCamera
local viewport =
    camera and camera.ViewportSize
    or Vector2.new(
        1366,
        768
    )
local sidebarWidth = 58
local sidebarHeight = 425
local sidebarFollowGap = 14
local sidebarSnapDistance = 90
local sidebarMagnetStrength = 0.28
local sidebar = Instance.new("Frame")
sidebar.Name =
    "Sidebar"
sidebar.Size =
    UDim2.fromOffset(
        sidebarWidth,
        sidebarHeight
    )
sidebar.Position =
    UDim2.fromOffset(
        viewport.X * 0.5 +
        baseWrapperSize.X * 0.5 +
        sidebarFollowGap,
        viewport.Y * 0.5
    )
sidebar.AnchorPoint =
    Vector2.new(
        0,
        0.5
    )
sidebar.BackgroundColor3 =
    Color3.fromRGB(
        91,
        28,
        140
    )
sidebar.BackgroundTransparency =
    0.2
sidebar.BorderSizePixel =
    0
sidebar.ZIndex =
    20
sidebar.Active =
    true
sidebar.Parent =
    gui
local sidebarCorner =
    Instance.new("UICorner")
sidebarCorner.CornerRadius =
    UDim.new(
        0,
        15
    )
sidebarCorner.Parent =
    sidebar
local sidebarStroke =
    Instance.new("UIStroke")
sidebarStroke.Color =
    Color3.fromRGB(
        205,
        120,
        255
    )
sidebarStroke.Thickness =
    1
sidebarStroke.Transparency =
    0.75
sidebarStroke.Parent =
    sidebar
local sidebarDragHandle =
    Instance.new("Frame")
sidebarDragHandle.Name =
    "DragHandle"
sidebarDragHandle.Size =
    UDim2.new(
        1,
        -10,
        0,
        18
    )
sidebarDragHandle.Position =
    UDim2.fromOffset(
        5,
        4
    )
sidebarDragHandle.BackgroundTransparency =
    1
sidebarDragHandle.BorderSizePixel =
    0
sidebarDragHandle.Active =
    true
sidebarDragHandle.ZIndex =
    25
sidebarDragHandle.Parent =
    sidebar
local dragGrip =
    Instance.new("Frame")
dragGrip.Name =
    "Grip"
dragGrip.Size =
    UDim2.fromOffset(
        24,
        3
    )
dragGrip.Position =
    UDim2.fromScale(
        0.5,
        0.5
    )
dragGrip.AnchorPoint =
    Vector2.new(
        0.5,
        0.5
    )
dragGrip.BackgroundColor3 =
    Color3.fromRGB(
        220,
        145,
        255
    )
dragGrip.BackgroundTransparency =
    0.45
dragGrip.BorderSizePixel =
    0
dragGrip.ZIndex =
    26
dragGrip.Parent =
    sidebarDragHandle
local dragGripCorner =
    Instance.new("UICorner")
dragGripCorner.CornerRadius =
    UDim.new(
        1,
        0
    )
dragGripCorner.Parent =
    dragGrip
local sidebarPadding =
    Instance.new("UIPadding")
sidebarPadding.PaddingTop =
    UDim.new(
        0,
        28
    )
sidebarPadding.PaddingBottom =
    UDim.new(
        0,
        12
    )
sidebarPadding.PaddingLeft =
    UDim.new(
        0,
        7
    )
sidebarPadding.PaddingRight =
    UDim.new(
        0,
        7
    )
sidebarPadding.Parent =
    sidebar
local sidebarLayout =
    Instance.new("UIListLayout")
sidebarLayout.HorizontalAlignment =
    Enum.HorizontalAlignment.Center
sidebarLayout.VerticalAlignment =
    Enum.VerticalAlignment.Top
sidebarLayout.Padding =
    UDim.new(
        0,
        9
    )
sidebarLayout.Parent =
    sidebar
local icons = {
    General = "rbxassetid://7733960981",
    Visual = "rbxassetid://7733774602",
    Misc = "rbxassetid://7734058803"
}
local buttons = {}
local function updateSelectedVisuals()
    for category, data in pairs(buttons) do
        local selected =
            category ==
            activeCategory
        TweenService:Create(
            data.button,
            TweenInfo.new(
                0.16,
                Enum.EasingStyle.Quart,
                Enum.EasingDirection.Out
            ),
            {
                BackgroundTransparency =
                    selected
                    and 0.58
                    or 1
            }
        ):Play()
        TweenService:Create(
            data.icon,
            TweenInfo.new(
                0.16,
                Enum.EasingStyle.Quart,
                Enum.EasingDirection.Out
            ),
            {
                ImageColor3 =
                    selected
                    and Color3.fromRGB(
                        255,
                        235,
                        255
                    )
                    or Color3.fromRGB(
                        210,
                        185,
                        225
                    ),
                ImageTransparency =
                    selected
                    and 0
                    or 0.08
            }
        ):Play()
        TweenService:Create(
            data.indicator,
            TweenInfo.new(
                0.16,
                Enum.EasingStyle.Quart,
                Enum.EasingDirection.Out
            ),
            {
                BackgroundTransparency =
                    selected
                    and 0
                    or 1
            }
        ):Play()
    end
end
local function createCategory(name)
    local button =
        Instance.new("TextButton")
    button.Name =
        name
    button.Size =
        UDim2.fromOffset(
            44,
            54
        )
    button.BackgroundColor3 =
        Color3.fromRGB(
            123,
            43,
            181
        )
    button.BackgroundTransparency =
        1
    button.BorderSizePixel =
        0
    button.Text =
        ""
    button.AutoButtonColor =
        false
    button.ZIndex =
        21
    button.Parent =
        sidebar
    local corner =
        Instance.new("UICorner")
    corner.CornerRadius =
        UDim.new(
            0,
            11
        )
    corner.Parent =
        button
    local icon =
        Instance.new("ImageLabel")
    icon.Name =
        "Icon"
    icon.Size =
        UDim2.fromOffset(
            23,
            23
        )
    icon.Position =
        UDim2.fromScale(
            0.5,
            0.5
        )
    icon.AnchorPoint =
        Vector2.new(
            0.5,
            0.5
        )
    icon.BackgroundTransparency =
        1
    icon.Image =
        icons[name]
    icon.ImageColor3 =
        Color3.fromRGB(
            210,
            185,
            225
        )
    icon.ImageTransparency =
        0.08
    icon.ZIndex =
        22
    icon.Parent =
        button
    local indicator =
        Instance.new("Frame")
    indicator.Name =
        "Indicator"
    indicator.Size =
        UDim2.fromOffset(
            3,
            24
        )
    indicator.Position =
        UDim2.new(
            0,
            -1,
            0.5,
            0
        )
    indicator.AnchorPoint =
        Vector2.new(
            0,
            0.5
        )
    indicator.BackgroundColor3 =
        Color3.fromRGB(
            220,
            145,
            255
        )
    indicator.BackgroundTransparency =
        1
    indicator.BorderSizePixel =
        0
    indicator.ZIndex =
        23
    indicator.Parent =
        button
    local indicatorCorner =
        Instance.new("UICorner")
    indicatorCorner.CornerRadius =
        UDim.new(
            1,
            0
        )
    indicatorCorner.Parent =
        indicator
    buttons[name] = {
        button = button,
        icon = icon,
        indicator = indicator
    }
    button.MouseEnter:Connect(function()
        if activeCategory ~= name then
            TweenService:Create(
                button,
                TweenInfo.new(
                    0.15,
                    Enum.EasingStyle.Quad,
                    Enum.EasingDirection.Out
                ),
                {
                    BackgroundTransparency =
                        0.72
                }
            ):Play()
            TweenService:Create(
                icon,
                TweenInfo.new(
                    0.15,
                    Enum.EasingStyle.Quad,
                    Enum.EasingDirection.Out
                ),
                {
                    ImageColor3 =
                        Color3.fromRGB(
                            240,
                            220,
                            250
                        )
                }
            ):Play()
        end
    end)
    button.MouseLeave:Connect(function()
        if activeCategory ~= name then
            TweenService:Create(
                button,
                TweenInfo.new(
                    0.15,
                    Enum.EasingStyle.Quad,
                    Enum.EasingDirection.Out
                ),
                {
                    BackgroundTransparency =
                        1
                }
            ):Play()
            TweenService:Create(
                icon,
                TweenInfo.new(
                    0.15,
                    Enum.EasingStyle.Quad,
                    Enum.EasingDirection.Out
                ),
                {
                    ImageColor3 =
                        Color3.fromRGB(
                            210,
                            185,
                            225
                        )
                }
            ):Play()
        end
    end)
    button.MouseButton1Click:Connect(function()
        activeCategory =
            name
        animateSection(
            name
        )
        updateSelectedVisuals()
        updateContentForCategory(
            name
        )
    end)
    return button
end
createCategory("General")
createCategory("Visual")
createCategory("Misc")
updateSelectedVisuals()
updateContentForCategory("General")
local sidebarDragging =
    false
local sidebarWasMoved =
    false
local sidebarAttached =
    false
local sidebarAttachedSide =
    "right"
local sidebarSnapCandidate =
    false
local sidebarSnapSide =
    nil
local sidebarDragStart
local sidebarStartPosition
local snapHelper =
    Instance.new("Frame")
snapHelper.Name =
    "SidebarSnapHelper"
snapHelper.Size =
    UDim2.fromOffset(
        sidebarWidth,
        sidebarHeight
    )
snapHelper.AnchorPoint =
    Vector2.new(
        0,
        0.5
    )
snapHelper.BackgroundColor3 =
    Color3.fromRGB(
        205,
        120,
        255
    )
snapHelper.BackgroundTransparency =
    1
snapHelper.BorderSizePixel =
    0
snapHelper.ZIndex =
    18
snapHelper.Visible =
    false
snapHelper.Parent =
    gui
local snapHelperCorner =
    Instance.new("UICorner")
snapHelperCorner.CornerRadius =
    UDim.new(
        0,
        15
    )
snapHelperCorner.Parent =
    snapHelper
local snapHelperStroke =
    Instance.new("UIStroke")
snapHelperStroke.Color =
    Color3.fromRGB(
        220,
        145,
        255
    )
snapHelperStroke.Thickness =
    2
snapHelperStroke.Transparency =
    1
snapHelperStroke.Parent =
    snapHelper
local snapConnection =
    Instance.new("Frame")
snapConnection.Name =
    "SnapConnection"
snapConnection.Size =
    UDim2.fromOffset(
        sidebarFollowGap,
        2
    )
snapConnection.AnchorPoint =
    Vector2.new(
        0,
        0.5
    )
snapConnection.BackgroundColor3 =
    Color3.fromRGB(
        220,
        145,
        255
    )
snapConnection.BackgroundTransparency =
    1
snapConnection.BorderSizePixel =
    0
snapConnection.ZIndex =
    19
snapConnection.Visible =
    false
snapConnection.Parent =
    gui
local snapConnectionCorner =
    Instance.new("UICorner")
snapConnectionCorner.CornerRadius =
    UDim.new(
        1,
        0
    )
snapConnectionCorner.Parent =
    snapConnection
local function getSidebarSnapPosition(side)
    local currentCamera =
        workspace.CurrentCamera
    if not currentCamera then
        return Vector2.zero
    end
    local currentViewport =
        currentCamera.ViewportSize
    local actualWrapperWidth =
        baseWrapperSize.X *
        currentStretchX *
        impactSquashX
    local halfWidth =
        actualWrapperWidth *
        0.5
    local center =
        Vector2.new(
            currentViewport.X *
            0.5 +
            currentX,
            currentViewport.Y *
            0.5 +
            currentY
        )
    if side == "left" then
        return Vector2.new(
            center.X -
            halfWidth -
            sidebarFollowGap -
            sidebarWidth,
            center.Y
        )
    end
    return Vector2.new(
        center.X +
        halfWidth +
        sidebarFollowGap,
        center.Y
    )
end
local function updateSnapVisual(side)
    local currentCamera =
        workspace.CurrentCamera
    if not currentCamera then
        return
    end
    local currentViewport =
        currentCamera.ViewportSize
    local actualWrapperWidth =
        baseWrapperSize.X *
        currentStretchX *
        impactSquashX
    local halfWidth =
        actualWrapperWidth *
        0.5
    local center =
        Vector2.new(
            currentViewport.X *
            0.5 +
            currentX,
            currentViewport.Y *
            0.5 +
            currentY
        )
    local target =
        getSidebarSnapPosition(
            side
        )
    snapHelper.Position =
        UDim2.fromOffset(
            target.X,
            target.Y
        )
    if side == "left" then
        snapConnection.Position =
            UDim2.fromOffset(
                center.X -
                halfWidth -
                sidebarFollowGap,
                center.Y
            )
    else
        snapConnection.Position =
            UDim2.fromOffset(
                center.X +
                halfWidth,
                center.Y
            )
    end
end
local function setSnapHelperVisible(state)
    if state then
        snapHelper.Visible =
            true
        snapConnection.Visible =
            true
        TweenService:Create(
            snapHelperStroke,
            TweenInfo.new(
                0.16,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                Transparency =
                    0.25
            }
        ):Play()
        TweenService:Create(
            snapHelper,
            TweenInfo.new(
                0.16,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                BackgroundTransparency =
                    0.88
            }
        ):Play()
        TweenService:Create(
            snapConnection,
            TweenInfo.new(
                0.16,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                BackgroundTransparency =
                    0.15
            }
        ):Play()
    else
        TweenService:Create(
            snapHelperStroke,
            TweenInfo.new(
                0.12,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                Transparency = 1
            }
        ):Play()
        TweenService:Create(
            snapHelper,
            TweenInfo.new(
                0.12,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                BackgroundTransparency = 1
            }
        ):Play()
        TweenService:Create(
            snapConnection,
            TweenInfo.new(
                0.12,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                BackgroundTransparency = 1
            }
        ):Play()
        task.delay(
            0.13,
            function()
                if not sidebarSnapCandidate then
                    snapHelper.Visible =
                        false
                    snapConnection.Visible =
                        false
                end
            end
        )
    end
end
sidebarDragHandle.InputBegan:Connect(function(input)
    if input.UserInputType ==
        Enum.UserInputType.MouseButton1 then
        sidebarDragging =
            true
        sidebarDragStart =
            input.Position
        sidebarStartPosition =
            sidebar.Position
        sidebarSnapCandidate =
            false
        sidebarSnapSide =
            nil
        setSnapHelperVisible(
            false
        )
        TweenService:Create(
            sidebarStroke,
            TweenInfo.new(
                0.12,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                Transparency =
                    0.35,
                Thickness =
                    2
            }
        ):Play()
        TweenService:Create(
            dragGrip,
            TweenInfo.new(
                0.12,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                BackgroundTransparency =
                    0
            }
        ):Play()
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if not sidebarDragging then
        return
    end
    if input.UserInputType ~=
        Enum.UserInputType.MouseMovement then
        return
    end
    local delta =
        input.Position -
        sidebarDragStart
    local freePosition =
        Vector2.new(
            sidebarStartPosition.X.Offset +
            delta.X,
            sidebarStartPosition.Y.Offset +
            delta.Y
        )
    if delta.Magnitude > 2 then
        sidebarWasMoved =
            true
        sidebarAttached =
            false
    end
    if not sidebarWasMoved then
        return
    end
    local rightTarget =
        getSidebarSnapPosition(
            "right"
        )
    local leftTarget =
        getSidebarSnapPosition(
            "left"
        )
    local rightDistance =
        (
            freePosition -
            rightTarget
        ).Magnitude
    local leftDistance =
        (
            freePosition -
            leftTarget
        ).Magnitude
    local nearestDistance =
        math.min(
            rightDistance,
            leftDistance
        )
    if nearestDistance <=
        sidebarSnapDistance then
        sidebarSnapCandidate =
            true
        if leftDistance <
            rightDistance then
            sidebarSnapSide =
                "left"
        else
            sidebarSnapSide =
                "right"
        end
        setSnapHelperVisible(
            true
        )
        updateSnapVisual(
            sidebarSnapSide
        )
        local snapTarget =
            sidebarSnapSide == "left"
            and leftTarget
            or rightTarget
        local normalizedDistance =
            math.clamp(
                nearestDistance /
                sidebarSnapDistance,
                0,
                1
            )
        local proximity =
            1 -
            normalizedDistance
        local strength =
            sidebarMagnetStrength +
            proximity *
            0.48
        strength =
            math.clamp(
                strength,
                0.15,
                0.76
            )
        local magneticPosition =
            freePosition +
            (
                snapTarget -
                freePosition
            ) *
            strength
        sidebar.Position =
            UDim2.fromOffset(
                magneticPosition.X,
                magneticPosition.Y
            )
    else
        sidebarSnapCandidate =
            false
        sidebarSnapSide =
            nil
        setSnapHelperVisible(
            false
        )
        sidebar.Position =
            UDim2.fromOffset(
                freePosition.X,
                freePosition.Y
            )
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType ~=
        Enum.UserInputType.MouseButton1 then
        return
    end
    if not sidebarDragging then
        return
    end
    sidebarDragging =
        false
    if sidebarSnapCandidate and
        sidebarSnapSide then
        local selectedSide =
            sidebarSnapSide
        sidebarAttached =
            true
        sidebarWasMoved =
            true
        local snapTarget =
            getSidebarSnapPosition(
                selectedSide
            )
        sidebarSnapCandidate =
            false
        sidebarSnapSide =
            nil
        sidebarAttachedSide =
            selectedSide
        setSnapHelperVisible(
            false
        )
        TweenService:Create(
            sidebar,
            TweenInfo.new(
                0.18,
                Enum.EasingStyle.Quart,
                Enum.EasingDirection.Out
            ),
            {
                Position =
                    UDim2.fromOffset(
                        snapTarget.X,
                        snapTarget.Y
                    )
            }
        ):Play()
    end
    TweenService:Create(
        sidebarStroke,
        TweenInfo.new(
            0.2,
            Enum.EasingStyle.Quart,
            Enum.EasingDirection.Out
        ),
        {
            Transparency =
                sidebarAttached
                and 0.55
                or 0.75,
            Thickness =
                sidebarAttached
                and 1.5
                or 1
        }
    ):Play()
    TweenService:Create(
        dragGrip,
        TweenInfo.new(
            0.2,
            Enum.EasingStyle.Quart,
            Enum.EasingDirection.Out
        ),
        {
            BackgroundTransparency =
                0.45
        }
    ):Play()
end)
menu.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local position = input.Position
        local outlineTrack = targetOutlineSlider and targetOutlineSlider.hitbox
        local fillTrack = targetFillSlider and targetFillSlider.hitbox
        local function inside(object)
            if not object or not object.Visible then
                return false
            end
            local absolutePosition = object.AbsolutePosition
            local absoluteSize = object.AbsoluteSize
            return position.X >= absolutePosition.X and position.X <= absolutePosition.X + absoluteSize.X and position.Y >= absolutePosition.Y and position.Y <= absolutePosition.Y + absoluteSize.Y
        end
        if inside(outlineTrack) or inside(fillTrack) then
            return
        end
    end
    if input.UserInputType ==
        Enum.UserInputType.MouseButton1 then
        dragging =
            true
        dragStart =
            input.Position
        startPosition =
            Vector2.new(
                wrapper.Position.X.Offset,
                wrapper.Position.Y.Offset
            )
        lastMousePosition =
            input.Position
        lastTime =
            os.clock()
        targetScale =
            1.025
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and
        input.UserInputType ==
        Enum.UserInputType.MouseMovement then
        local now =
            os.clock()
        local dt =
            math.max(
                now -
                lastTime,
                0.001
            )
        local mouseDelta =
            input.Position -
            lastMousePosition
        local delta =
            input.Position -
            dragStart
        targetX =
            startPosition.X +
            delta.X
        targetY =
            startPosition.Y +
            delta.Y
        velocityX =
            mouseDelta.X /
            dt
        velocityY =
            mouseDelta.Y /
            dt
        local speed =
            math.sqrt(
                velocityX *
                velocityX +
                velocityY *
                velocityY
            )
        local normalizedSpeed =
            math.clamp(
                speed /
                1800,
                0,
                1
            )
        targetRotation =
            math.clamp(
                velocityX *
                0.018,
                -8,
                8
            )
        local horizontalStretch =
            math.clamp(
                math.abs(
                    velocityX
                ) /
                2500,
                0,
                0.055
            )
        local verticalStretch =
            math.clamp(
                math.abs(
                    velocityY
                ) /
                3000,
                0,
                0.035
            )
        targetStretchX =
            1 +
            horizontalStretch
        targetStretchY =
            1 -
            horizontalStretch *
            0.55
        if math.abs(
            velocityY
        ) >
            math.abs(
                velocityX
            ) then
            targetStretchY =
                1 +
                verticalStretch
            targetStretchX =
                1 -
                verticalStretch *
                0.5
        end
        targetScale =
            1.025 +
            normalizedSpeed *
            0.018
        lastMousePosition =
            input.Position
        lastTime =
            now
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType ==
        Enum.UserInputType.MouseButton1 then
        dragging =
            false
        targetRotation =
            0
        targetScale =
            1
        targetStretchX =
            1
        targetStretchY =
            1
        velocityX =
            velocityX *
            0.35
        velocityY =
            velocityY *
            0.35
    end
end)
local ghostTimer =
    0
RunService.RenderStepped:Connect(function(dt)
    if not gui.Parent then
        return
    end
    local positionSmooth =
        math.clamp(
            dt * 11,
            0,
            1
        )
    local rotationSmooth =
        math.clamp(
            dt * 10,
            0,
            1
        )
    local scaleSmooth =
        math.clamp(
            dt * 12,
            0,
            1
        )
    local stretchSmooth =
        math.clamp(
            dt * 13,
            0,
            1
        )
    if not dragging then
        targetX =
            targetX +
            velocityX *
            dt
        targetY =
            targetY +
            velocityY *
            dt
        handleWalls()
        local friction =
            math.pow(
                0.035,
                dt
            )
        velocityX =
            velocityX *
            friction
        velocityY =
            velocityY *
            friction
    end
    overshootX =
        overshootX +
        overshootVelocityX *
        dt
    overshootY =
        overshootY +
        overshootVelocityY *
        dt
    local overshootFriction =
        math.pow(
            0.025,
            dt
        )
    overshootVelocityX =
        overshootVelocityX *
        overshootFriction
    overshootVelocityY =
        overshootVelocityY *
        overshootFriction
    overshootX =
        overshootX *
        math.pow(
            0.12,
            dt
        )
    overshootY =
        overshootY *
        math.pow(
            0.12,
            dt
        )
    impactSquashX =
        impactSquashX +
        (
            1 -
            impactSquashX
        ) *
        math.clamp(
            dt * 9,
            0,
            1
        )
    impactSquashY =
        impactSquashY +
        (
            1 -
            impactSquashY
        ) *
        math.clamp(
            dt * 9,
            0,
            1
        )
    currentScale =
        currentScale +
        (
            targetScale -
            currentScale
        ) *
        scaleSmooth
    currentStretchX =
        currentStretchX +
        (
            targetStretchX -
            currentStretchX
        ) *
        stretchSmooth
    currentStretchY =
        currentStretchY +
        (
            targetStretchY -
            currentStretchY
        ) *
        stretchSmooth
    currentX =
        currentX +
        (
            targetX +
            overshootX -
            currentX
        ) *
        positionSmooth
    currentY =
        currentY +
        (
            targetY +
            overshootY -
            currentY
        ) *
        positionSmooth
    wrapper.Position =
        UDim2.new(
            0.5,
            currentX,
            0.5,
            currentY
        )
    currentRotation =
        currentRotation +
        (
            targetRotation -
            currentRotation
        ) *
        rotationSmooth
    wrapper.Rotation =
        currentRotation
    scale.Scale =
        currentScale
    menu.Size =
        UDim2.fromOffset(
            baseMenuSize.X *
            currentStretchX *
            impactSquashX,
            baseMenuSize.Y *
            currentStretchY *
            impactSquashY
        )
    wrapper.Size =
        UDim2.fromOffset(
            baseWrapperSize.X *
            currentStretchX *
            impactSquashX,
            baseWrapperSize.Y *
            currentStretchY *
            impactSquashY
        )
    if not sidebarDragging then
        if sidebarAttached then
            local attachedTarget =
                getSidebarSnapPosition(
                    sidebarAttachedSide
                )
            sidebar.Position =
                UDim2.fromOffset(
                    attachedTarget.X,
                    attachedTarget.Y
                )
        elseif not sidebarWasMoved then
            local followTarget =
                getSidebarSnapPosition(
                    "right"
                )
            sidebar.Position =
                UDim2.fromOffset(
                    followTarget.X,
                    followTarget.Y
                )
        end
    end
    if sidebarSnapCandidate and
        sidebarSnapSide then
        updateSnapVisual(
            sidebarSnapSide
        )
    end
    local speed =
        math.sqrt(
            velocityX *
            velocityX +
            velocityY *
            velocityY
        )
    local movement =
        math.clamp(
            speed /
            1800,
            0,
            1
        )
    menuStroke.Transparency =
        0.82 -
        movement *
        0.5
    wrapperStroke.Transparency =
        0.8 -
        movement *
        0.25
    menuStroke.Thickness =
        1 +
        movement *
        2
    wrapperStroke.Thickness =
        1 +
        movement *
        1.5
    if not sidebarDragging then
        sidebarStroke.Transparency =
            sidebarAttached
            and 0.55
            or (
                0.75 -
                movement *
                0.2
            )
    end
    ghostTimer =
        ghostTimer +
        dt
    if dragging and
        ghostTimer >= 0.035 then
        ghostTimer =
            0
        updateGhosts()
    end
    for i = 1, ghostCount do
        local data =
            ghosts[i]
        data.alpha =
            data.alpha *
            math.pow(
                0.78,
                dt * 60
            )
        local ghost =
            data.frame
        local stroke =
            data.stroke
        ghost.Position =
            UDim2.new(
                0.5,
                data.x,
                0.5,
                data.y
            )
        ghost.Rotation =
            data.rotation
        local distance =
            math.sqrt(
                (
                    currentX -
                    data.x
                ) ^ 2 +
                (
                    currentY -
                    data.y
                ) ^ 2
            )
        local ghostScale =
            data.scale +
            math.clamp(
                distance /
                1500,
                0,
                0.035
            )
        ghost.Size =
            UDim2.fromOffset(
                baseMenuSize.X *
                ghostScale,
                baseMenuSize.Y *
                ghostScale
            )
        local fadeMultiplier =
            1 -
            (
                (i - 1) /
                ghostCount
            )
        ghost.BackgroundTransparency =
            1 -
            math.clamp(
                data.alpha *
                fadeMultiplier,
                0,
                0.58
            )
        stroke.Transparency =
            1 -
            math.clamp(
                data.alpha *
                fadeMultiplier *
                1.7,
                0,
                0.8
            )
    end
end)
local menuVisible = true
local closing =
    false
local function setMenuVisible(visible)
    if closing then
        return
    end
    menuVisible = visible
    wrapper.Visible = visible
    ghostLayer.Visible = visible
    sidebar.Visible = visible
    if not visible then
        hideTargetColorPaletteForMenu()
    elseif activeCategory == "Visual" and targetHighlightEnabled and targetColorPaletteOpenState then
        openTargetColorPalette()
    end
    if not visible then
        setSnapHelperVisible(false)
    else
        snapHelper.Visible = false
        snapConnection.Visible = false
    end
    TweenService:Create(
        blur,
        TweenInfo.new(
            0.15,
            Enum.EasingStyle.Quad,
            Enum.EasingDirection.Out
        ),
        {
            Size = visible and 10 or 0
        }
    ):Play()
end
local function closeMenu()
    if closing then
        return
    end
    closing =
        true
    destroyVisualCube()
    setTargetHighlightEnabled(false)
    setAimbotMouseEnabled(false)
    local fade =
        TweenService:Create(
            blur,
            TweenInfo.new(
                0.15,
                Enum.EasingStyle.Quad,
                Enum.EasingDirection.Out
            ),
            {
                Size = 0
            }
        )
    fade:Play()
    fade.Completed:Connect(function()
        if blur then
            blur:Destroy()
        end
        if gui then
            gui:Destroy()
        end
    end)
end
UserInputService.InputBegan:Connect(function(
    input,
    gameProcessed
)
    if gameProcessed then
        return
    end
    if input.KeyCode ==
        Enum.KeyCode.Delete then
        setMenuVisible(
            not menuVisible
        )
        return
    end
    if input.KeyCode ==
        Enum.KeyCode.P then
        closeMenu()
    end
end)
player.CharacterAdded:Connect(function()
    if visualEnabled then
        task.wait(0.2)
        createVisualCube()
    end
    if aimbotMouseEnabled then
        task.wait(0.2)
        local character =
            player.Character
        aimbotHumanoid =
            character and
            character:FindFirstChildOfClass(
                "Humanoid"
            )
            or nil
        if aimbotHumanoid then
            aimbotHumanoid.AutoRotate =
                false
        end
    end
end)
