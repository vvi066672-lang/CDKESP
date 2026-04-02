# CDKESP -- CaoDangKhoi v1 PRO MAX FULL - HOÀN CHIỀU
local P=game:GetService("Players")
local L=P.LocalPlayer
local C=workspace.CurrentCamera
local R=game:GetService("RunService")
local U=game:GetService("UserInputService")
local T=game:GetService("TweenService")

-- CONFIG
local cfg={fov=140,smooth=.18,lock=.35,part="Head",auto=false,mode="smooth",predict=.12}
local esp={box=true,name=true,skel=true,hp=true,radar=true,hl=true}

-- GUI
local g=Instance.new("ScreenGui",L.PlayerGui)
g.ResetOnSpawn=false
local f=Instance.new("Frame",g)
f.Size=UDim2.new(0,260,0,380)
f.Position=UDim2.new(.1,0,.3,0)
f.BackgroundColor3=Color3.fromRGB(8,8,8)
f.Active=true
f.Draggable=true
Instance.new("UICorner",f)
Instance.new("UIStroke",f).Color=Color3.fromRGB(0,255,0)

-- LOGO
local logo=Instance.new("ImageButton",g)
logo.Size=UDim2.new(0,65,0,65)
logo.Position=UDim2.new(0,20,1,-100)
logo.Image="rbxassetid://3926305904"
logo.BackgroundColor3=Color3.fromRGB(0,0,0)
Instance.new("UICorner",logo).CornerRadius=UDim.new(1,0)
local stroke=Instance.new("UIStroke",logo)
stroke.Color=Color3.fromRGB(0,255,80)
stroke.Thickness=3
local spinInfo=TweenInfo.new(.9,Enum.EasingStyle.Linear)
local function spin() T:Create(logo,spinInfo,{Rotation=logo.Rotation+360}):Play() end
logo.MouseButton1Click:Connect(function() f.Visible=not f.Visible; spin() end)
task.spawn(function() while true do for i=0,1,.05 do stroke.Transparency=.7-i*.7; task.wait() end for i=1,0,-.05 do stroke.Transparency=.7-i*.7; task.wait() end end end)

-- BUTTON
local function b(t,y)
    local btn=Instance.new("TextButton",f)
    btn.Size=UDim2.new(1,-20,0,26)
    btn.Position=UDim2.new(0,10,0,y)
    btn.Text=t
    btn.BackgroundColor3=Color3.fromRGB(15,15,15)
    btn.TextColor3=Color3.fromRGB(0,255,0)
    Instance.new("UICorner",btn)
    return btn
end

local title=Instance.new("TextLabel",f)
title.Size=UDim2.new(1,0,0,30)
title.BackgroundTransparency=1
title.Text="CaoDangKhoi v1 PRO MAX"
title.TextColor3=Color3.fromRGB(0,255,0)
title.Font=Enum.Font.GothamBlack

-- BUTTON LIST
local aimBtn=b("AIM: TẮT",35)
local modeBtn=b("KIỂU AIM: MƯỢT",65)
local partBtn=b("MỤC TIÊU: ĐẦU",95)
local autoBtn=b("TỰ BẮN: TẮT",125)
local boxBtn=b("ESP HỘP: BẬT",155)
local nameBtn=b("ESP TÊN: BẬT",185)
local skelBtn=b("ESP XƯƠNG: BẬT",215)
local hpBtn=b("HIỆN MÁU: BẬT",245)
local radarBtn=b("RADAR: BẬT",275)
local hlBtn=b("HIGHLIGHT: BẬT",305)

-- FOV
local FOV=Drawing.new("Circle")
FOV.Color=Color3.fromRGB(0,255,0)
FOV.Thickness=2
FOV.Filled=false

-- RADAR
local radar=Instance.new("Frame",g)
radar.Size=UDim2.new(0,120,0,120)
radar.Position=UDim2.new(1,-130,0,20)
radar.BackgroundColor3=Color3.fromRGB(0,0,0)
Instance.new("UIStroke",radar).Color=Color3.fromRGB(0,255,0)

-- STATE
local aim=false
local highlights={}
local dots={}
local drawings={}
local skeletons={}
local bones={
{"Head","UpperTorso"},{"UpperTorso","LowerTorso"},{"UpperTorso","LeftUpperArm"},{"LeftUpperArm","LeftLowerArm"},{"LeftLowerArm","LeftHand"},
{"UpperTorso","RightUpperArm"},{"RightUpperArm","RightLowerArm"},{"RightLowerArm","RightHand"},
{"LowerTorso","LeftUpperLeg"},{"LeftUpperLeg","LeftLowerLeg"},{"LeftLowerLeg","LeftFoot"},
{"LowerTorso","RightUpperLeg"},{"RightUpperLeg","RightLowerLeg"},{"RightLowerLeg","RightFoot"}
}

-- CHECK
local function alive(p)
    return p and p.Character and p.Character:FindFirstChild("Humanoid") and p.Character:FindFirstChild("HumanoidRootPart") and p.Character.Humanoid.Health>0
end

-- CREATE ESP
local function createESP(p)
    drawings[p]={
        box=Drawing.new("Square"),
        name=Drawing.new("Text"),
        hp=Drawing.new("Line")
    }
    drawings[p].box.Color=Color3.fromRGB(0,255,0)
    drawings[p].box.Filled=false
    drawings[p].name.Size=13
    drawings[p].name.Center=true
    drawings[p].name.Outline=true
    drawings[p].hp.Thickness=2
    skeletons[p]={}
    for i=1,#bones do
        local l=Drawing.new("Line")
        l.Color=Color3.fromRGB(0,255,0)
        table.insert(skeletons[p],l)
    end
end

-- SETUP CHAR
local function setupChar(p,char)
    if highlights[p] then highlights[p]:Destroy() end
    local h=Instance.new("Highlight")
    h.FillTransparency=1
    h.OutlineColor=Color3.fromRGB(0,255,0)
    h.Parent=char
    highlights[p]=h
    local dot=Instance.new("Frame")
    dot.Size=UDim2.new(0,4,0,4)
    dot.BackgroundColor3=Color3.fromRGB(0,255,0)
    dot.Parent=radar
    dots[p]=dot
end

-- SETUP PLAYER
local function setupPlayer(p)
    if p==L then return end
    createESP(p)
    if p.Character then setupChar(p,p.Character) end
    p.CharacterAdded:Connect(function(c) task.wait(.3) setupChar(p,c) end)
end

for _,p in pairs(P:GetPlayers()) do setupPlayer(p) end
P.PlayerAdded:Connect(setupPlayer)

-- PREDICT
local function predict(part)
    return part.Position+(part.Velocity*cfg.predict)
end

-- GET PART
local function getPart(c)
    local part=c:FindFirstChild(cfg.part) or c:FindFirstChild("HumanoidRootPart")
    return part,predict(part)
end

-- CLOSEST
local function closest()
    local best=nil
    local dist=math.huge
    for _,p in pairs(P:GetPlayers()) do
        if p~=L and alive(p) then
            local part,_=getPart(p.Character)
            local v,vis=C:WorldToViewportPoint(part.Position)
            if vis then
                local diff=(Vector2.new(v.X,v.Y)-FOV.Position).Magnitude
                if diff<cfg.fov and diff<dist then dist=diff; best=p end
            end
        end
    end
    return best
end

-- HOTKEY
U.InputBegan:Connect(function(i,gp)
    if not gp and i.KeyCode==Enum.KeyCode.Insert then f.Visible=not f.Visible; spin() end
end)

-- BUTTON TOGGLE
aimBtn.MouseButton1Click:Connect(function() aim=not aim; aimBtn.Text="AIM: "..(aim and "BẬT" or "TẮT") end)
modeBtn.MouseButton1Click:Connect(function() cfg.mode=cfg.mode=="smooth" and "lock" or "smooth"; modeBtn.Text="KIỂU AIM: "..(cfg.mode=="smooth" and "MƯỢT" or "CHẶT") end)
partBtn.MouseButton1Click:Connect(function() cfg.part=cfg.part=="Head" and "HumanoidRootPart" or "Head"; partBtn.Text="MỤC TIÊU: "..(cfg.part=="Head" and "ĐẦU" or "THÂN") end)
autoBtn.MouseButton1Click:Connect(function() cfg.auto=not cfg.auto; autoBtn.Text="TỰ BẮN: "..(cfg.auto and "BẬT" or "TẮT") end)
boxBtn.MouseButton1Click:Connect(function() esp.box=not esp.box; boxBtn.Text="ESP HỘP: "..(esp.box and "BẬT" or "TẮT") end)
nameBtn.MouseButton1Click:Connect(function() esp.name=not esp.name; nameBtn.Text="ESP TÊN: "..(esp.name and "BẬT" or "TẮT") end)
skelBtn.MouseButton1Click:Connect(function() esp.skel=not esp.skel; skelBtn.Text="ESP XƯƠNG: "..(esp.skel and "BẬT" or "TẮT") end)
hpBtn.MouseButton1Click:Connect(function() esp.hp=not esp.hp; hpBtn.Text="HIỆN MÁU: "..(esp.hp and "BẬT" or "TẮT") end)
radarBtn.MouseButton1Click:Connect(function() esp.radar=not esp.radar; radar.Visible=esp.radar; radarBtn.Text="RADAR: "..(esp.radar and "BẬT" or "TẮT") end)
hlBtn.MouseButton1Click:Connect(function() esp.hl=not esp.hl; hlBtn.Text="HIGHLIGHT: "..(esp.hl and "BẬT" or "TẮT"); for _,h in pairs(highlights) do h.Enabled=esp.hl end end)

-- LOOP
R.RenderStepped:Connect(function()
    FOV.Radius=cfg.fov
    FOV.Position=Vector2.new(C.ViewportSize.X/2,C.ViewportSize.Y/2)

    -- RADAR
    if L.Character and L.Character:FindFirstChild("HumanoidRootPart") then
        for p,dot in pairs(dots) do
            if alive(p) then
                local rel=(p.Character.HumanoidRootPart.Position-L.Character.HumanoidRootPart.Position)/7
                dot.Position=UDim2.new(.5,rel.X,.5,rel.Z)
                dot.Visible=esp.radar
            else dot.Visible=false end
        end
    end

    -- ESP BOX NAME HP
    for p,d in pairs(drawings) do
        if alive(p) then
            local root=p.Character.HumanoidRootPart
            local pos,vis=C:WorldToViewportPoint(root.Position)
            if vis then
                local size=(C:WorldToViewportPoint(root.Position+Vector3.new(0,3,0)).Y-pos.Y)*2
                d.box.Visible=esp.box
                d.box.Size=Vector2.new(size,size*1.6)
                d.box.Position=Vector2.new(pos.X-size/2,pos.Y-size*1.3)
                d.name.Visible=esp.name
                d.name.Position=Vector2.new(pos.X,pos.Y-size*1.5)
                d.name.Text=p.Name
                local hp=p.Character.Humanoid.Health/p.Character.Humanoid.MaxHealth
                d.hp.Visible=esp.hp
                d.hp.From=Vector2.new(pos.X-size/2-6,pos.Y+size*.8)
                d.hp.To=Vector2.new(pos.X-size/2-6,pos.Y+size*.8-(size*1.6*hp))
                d.hp.Color=Color3.fromRGB(255-255*hp,255*hp,0)
            else d.box.Visible=false; d.name.Visible=false; d.hp.Visible=false end
        else d.box.Visible=false; d.name.Visible=false; d.hp.Visible=false end
    end

    -- SKELETON
    for p,lines in pairs(skeletons) do
        if alive(p) then
            local char=p.Character
            for i,b in pairs(bones) do
                local p1=char:FindFirstChild(b[1])
                local p2=char:FindFirstChild(b[2])
                local l=lines[i]
                if p1 and p2 then
                    local v1,vis1=C:WorldToViewportPoint(p1.Position)
                    local v2,vis2=C:WorldToViewportPoint(p2.Position)
                    if vis1 and vis2 and esp.skel then
                        l.Visible=true
                        l.From=Vector2.new(v1.X,v1.Y)
                        l.To=Vector2.new(v2.X,v2.Y)
                    else l.Visible=false end
                else l.Visible=false end
            end
        else for _,l in pairs(lines) do l.Visible=false end
        end
    end

    -- AIM
    local target=closest()
    if aim and target and alive(target) then
        local part,pred=getPart(target.Character)
        local goal=CFrame.new(C.CFrame.Position,pred or part.Position)
        local power=(cfg.mode=="smooth") and cfg.smooth or cfg.lock
        C.CFrame=C.CFrame:Lerp(goal,power)
    end
end)
