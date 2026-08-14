-- ========================================================
-- GREEDY GROWERS - SMART AUTO HARVEST & EARLY WARNING V3 (FIXED)
-- ========================================================
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({ 
    Name = "⚡ Greedy Growers - Fix Triệt Để", 
    LoadingTitle = "Đang khởi động hệ thống tối ưu...", 
    LoadingSubtitle = "By Gemini",
    KeySystem = false 
})

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

local Tab = Window:CreateTab("⚡ Điều Khiển & Cảnh Báo", 4483362458)

local StatusLabel = Tab:CreateLabel("⏱️ Trạng thái: Đang sẵn sàng...")
local isEarlyWarningEnabled = false
local isAutoHarvestEnabled = false

Tab:CreateToggle({
   Name = "🚨 Bật Cảnh Báo Sét (Đã fix bộ lọc)",
   CurrentValue = false,
   Callback = function(v)
      isEarlyWarningEnabled = v
      if v then
          Rayfield:Notify({Title = "Đã bật!", Content = "Hệ thống canh chừng đã hoạt động.", Duration = 2})
          StatusLabel:Set("⏱️ Trạng thái: Đang theo dõi môi trường...")
      else
          StatusLabel:Set("⏱️ Trạng thái: Đã tắt")
      end
   end,
})

Tab:CreateToggle({
   Name = "⚡ Tự Động Thu Hoạch (Auto Harvest)",
   CurrentValue = false,
   Callback = function(v)
      isAutoHarvestEnabled = v
      if v then
          Rayfield:Notify({Title = "Auto Harvest: Bật", Content = "Sẵn sàng gom cây khi có lệnh.", Duration = 2})
      else
          Rayfield:Notify({Title = "Auto Harvest: Tắt", Content = "Đã tắt tính năng tự động.", Duration = 2})
      end
   end,
})

-- Hàm kích hoạt thu hoạch tối ưu hóa quét ProximityPrompt toàn diện
local function triggerHarvest()
    pcall(function()
        local character = LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return end
        local hrp = character.HumanoidRootPart

        -- Quét tất cả Prompt trong game thay vì kén chọn tên
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                -- Kiểm tra khoảng cách để tránh tương tác với mấy thứ ở xa tít tắp
                local parentPart = obj.Parent
                if parentPart and parentPart:IsA("BasePart") then
                    if (parentPart.Position - hrp.Position).Magnitude <= 30 then
                        fireproximityprompt(obj)
                    end
                end
            end
        end
    end)
end

-- Vòng lặp giám sát cải tiến không bỏ sót
task.spawn(function()
    local lastAlertTime = 0
    while true do
        task.wait(0.5) -- Tăng độ ổn định, giảm tải cho game
        if isEarlyWarningEnabled or isAutoHarvestEnabled then
            pcall(function()
                local triggered = false
                
                -- Phương thức mới: Kiểm tra trực tiếp các thay đổi về hiệu ứng âm thanh/part sấm sét thực tế trong Workspace
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("Sound") and (obj.SoundId:find("storm") or obj.SoundId:find("thunder") or obj.SoundId:find("lightning")) and obj.Playing then
                        triggered = true
                        break
                    elseif obj:IsA("ParticleEmitter") and (obj.Name:lower():find("rain") or obj.Name:lower():find("storm")) and obj.Enabled then
                        triggered = true
                        break
                    end
                end

                if triggered and (os.time() - lastAlertTime > 10) then
                    lastAlertTime = os.time()
                    
                    if isAutoHarvestEnabled then
                        StatusLabel:Set("⚡ ĐÃ KÍCH HOẠT THU HOẠCH ĐÓN ĐẦU!")
                        Rayfield:Notify({
                            Title = "⚡ AUTO HARVEST!",
                            Content = "Đã tự động tương tác để gom cây thành công!",
                            Duration = 3,
                            Image = 4483362458,
                        })
                        triggerHarvest()
                    elseif isEarlyWarningEnabled then
                        StatusLabel:Set("⚠️ CẢNH BÁO: PHÁT HIỆN SẤM SÉT!")
                        Rayfield:Notify({
                            Title = "⚠️ BÁO ĐỘNG ĐỎ!",
                            Content = "Thu hoạch ngay lập tức!",
                            Duration = 3,
                            Image = 4483362458,
                        })
                    end
                    
                    task.delay(3, function()
                        if isEarlyWarningEnabled or isAutoHarvestEnabled then
                            StatusLabel:Set("⏱️ Trạng thái: Tiếp tục canh chừng...")
                        end
                    end)
                end
            end)
        end
    end
end)
