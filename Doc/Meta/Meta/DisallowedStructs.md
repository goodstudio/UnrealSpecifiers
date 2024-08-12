# DisallowedStructs

Usage: UPROPERTY
Feature: Blueprint
Type: string="abc"
Description: 只在SmartObject模块中应用，用以在类选择器中排除掉某个类以及子类。
Status: OnlyInternal
Group: TypePicker

只在SmartObject模块中应用，用以在类选择器中排除掉某个类以及子类。

源码：

```cpp
UPROPERTY(EditDefaultsOnly, Category = "SmartObject", meta=(DisallowedStructs="/Script/SmartObjectsModule.SmartObjectSlotAnnotation"))
TArray<FSmartObjectDefinitionDataProxy> DefinitionData;
```