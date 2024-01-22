# Wrapper
Generated by [Rojo](https://github.com/rojo-rbx/rojo) 7.3.0.

## Wally Installation
To install with wally, insert it above wally.toml [dependecies]
```toml
Wrapper = "ernisto/wrapper@0.4.4"
```

## Usage
```lua
type data = { name: string, maxHealth: number }

function Minion.deserialize(data: data)
    
    local asset = MinionModels[data.name]
    local model = asset:Clone()
    
    local self = wrapper(model, 'Minion')   -- add tag Minion to model
        :_syncAttributes(data)  -- all fields of `data` can be read and written from `self`, all fields will appear in `model` as attributes, and modifications from `self` will be applied to `data`
    
    self:addTags('loading')
    print(self:is('Minion', 'loading'))    --> true, 
    
    self.health = self.maxHealth -- automaticly defines a attribute into `model` with value of `self.maxHealth`
    self.targetPosition = Instance.new("Attachment", WORLD_ROOT)    -- automaticly creates a ObjectValue into `model`
    
    local onDie = self:_host(Instance.new("BindableEvent"))  -- will define BindableEvent.Parent to `model` if havent .Parent, and when `self` be :unwrap'ed, the bindable will be :Destroy'ed
    self.onDie = onDie.Event
    
    --// Cleaner
    local cancelClean = self:cleaner(function() -- set a callback when `self` be :unwrap'ed, returns a canceller for convenience
        
        self.onDie.Event:Fire()
    end)
    
    --// Methods
    function self:takeDamage(damage: number)
        
        self.health -= damage   -- automaticly update the 'health' attribute
        if self.health <= 0 then self:destroy() end -- automaticly model:Destroy() and self:unwrap()
    end
    function self:upgradeHealth()
        
        self.maxHealth *= 1.10  -- automaticly update the 'maxHealth' attribute, and update `self.maxHealth`
    end
    function self:moveTo(cframe: CFrame, alignedto: BasePart)
        
        self.targetPosition = Instance.new("Attachment", alginedto or WORLD_ROOT)    -- automaticly update the ObjectValue of `self.targetPosition`
        self.targetPosition.CFrame = cframe
    end
    
    --// Loop
    local movement = RunService.Heartbeat:Connect(function()
    end)
    self:_host(movement)    -- when `self` be :unwrap'ed, the movement will be :Disconnect'ed
    
    return self
end
```