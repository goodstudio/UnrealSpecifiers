# CollapsableChildProperties

Usage: UPROPERTY
Feature: Editor
Type: bool
LimitedType: TextureGraph插件内使用
Status: LimitedInPlugin
Parent item: ShowInnerProperties (ShowInnerProperties.md)

在TextureGraph模块中新增加的meta。用于折叠一个结构的内部属性。

源码：

```cpp
bool STG_GraphPinOutputSettings::CollapsibleChildProperties() const
{
	FProperty* Property = GetPinProperty();
	bool Collapsible = false;
	// check if there is a display name defined for the property, we use that as the Pin Name
	if (Property && Property->HasMetaData("CollapsableChildProperties"))
	{
		Collapsible = true;
	}
	return Collapsible;
}

	UPROPERTY(EditAnywhere, Category = NoCategory, meta = (TGType = "TG_Input", CollapsableChildProperties,ShowOnlyInnerProperties, FullyExpand, NoResetToDefault, PinDisplayName = "Settings") )
	FTG_OutputSettings OutputSettings;
```