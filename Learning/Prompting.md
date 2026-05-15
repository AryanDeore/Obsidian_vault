1. **Conditioning**: Provide additional context to steer the model in a particular direction.
2. Assign role and responsibility: Claude, you are responsible for identifying harmful aspects in prompts. Is this image generation prompt safe?
3. **Structured Output**: XML tags. to reliably follow instructions and generate reliable outputs
```XML
<description>
The SmartHome Mini is a compact smart home assistant available in black or white for 
only $49.99. At just 5 inches wide, it lets you control lights, thermostats, and other 
connected devices via voice or app—no matter where you place it in your home. This 
affordable little hub brings convenient hands-free control to your smart devices.
</description>

1. Extract the <name>, <size>, <price>, and <color> from this product <description>.

2. Provide <summary> of the <reviews> corresponding to <id>

3. extract the <aspect>, <sentiment>, and corresponding review <id>
```

