# HasDedicatedAsyncNode

- **Usage Location:** UCLASS
- **Metadata Type:** bool
- **Associated Items:** [ExposedAsyncProxy](../ExposedAsyncProxy/ExposedAsyncProxy.md)

Hide the blueprint asynchronous node automatically generated by the factory method in the UBlueprintAsyncActionBase subclass, allowing for manual creation of a corresponding UK2Node_XXX.

```cpp
/**
* BlueprintCallable factory functions for classes which inherit from UBlueprintAsyncActionBase will have a special blueprint node created for it: UK2Node_AsyncAction
* You can stop this node spawning and create a more specific one by adding the UCLASS metadata "HasDedicatedAsyncNode"
*/

UCLASS(MinimalAPI)
class UBlueprintAsyncActionBase : public UObject
{}
```

## Test Code:

```cpp
UCLASS(Blueprintable, BlueprintType,meta = (ExposedAsyncProxy = MyAsyncObject,HasDedicatedAsyncNode))
class INSIDER_API UMyFunction_Async :public UCancellableAsyncAction
{}

//You can customize a K2Node
UCLASS()
class INSIDER_API UK2Node_MyFunctionAsyncAction : public UK2Node_AsyncAction
{
	GENERATED_BODY()

	// UK2Node interface
	virtual void GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const override;
	virtual void AllocateDefaultPins() override;
	// End of UK2Node interface

protected:
	virtual bool HandleDelegates(
		const TArray<FBaseAsyncTaskHelper::FOutputPinAndLocalVariable>& VariableOutputs, UEdGraphPin* ProxyObjectPin,
		UEdGraphPin*& InOutLastThenPin, UEdGraph* SourceGraph, FKismetCompilerContext& CompilerContext) override;
};

void UK2Node_MyFunctionAsyncAction::GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const
{
	struct GetMenuActions_Utils
	{
		static void SetNodeFunc(UEdGraphNode* NewNode, bool /*bIsTemplateNode*/, TWeakObjectPtr<UFunction> FunctionPtr)
		{
			UK2Node_MyFunctionAsyncAction* AsyncTaskNode = CastChecked<UK2Node_MyFunctionAsyncAction>(NewNode);
			if (FunctionPtr.IsValid())
			{
				UFunction* Func = FunctionPtr.Get();
				FObjectProperty* ReturnProp = CastFieldChecked<FObjectProperty>(Func->GetReturnProperty());

				AsyncTaskNode->ProxyFactoryFunctionName = Func->GetFName();
				AsyncTaskNode->ProxyFactoryClass = Func->GetOuterUClass();
				AsyncTaskNode->ProxyClass = ReturnProp->PropertyClass;
				AsyncTaskNode->NodeComment = TEXT("This is MyCustomK2Node");
			}
		}
	};

	UClass* NodeClass = GetClass();
	ActionRegistrar.RegisterClassFactoryActions<UMyFunction_Async>(FBlueprintActionDatabaseRegistrar::FMakeFuncSpawnerDelegate::CreateLambda([NodeClass](const UFunction* FactoryFunc)->UBlueprintNodeSpawner*
		{
			UBlueprintNodeSpawner* NodeSpawner = UBlueprintFunctionNodeSpawner::Create(FactoryFunc);
			check(NodeSpawner != nullptr);
			NodeSpawner->NodeClass = NodeClass;

			TWeakObjectPtr<UFunction> FunctionPtr = MakeWeakObjectPtr(const_cast<UFunction*>(FactoryFunc));
			NodeSpawner->CustomizeNodeDelegate = UBlueprintNodeSpawner::FCustomizeNodeDelegate::CreateStatic(GetMenuActions_Utils::SetNodeFunc, FunctionPtr);

			return NodeSpawner;
		}));
}

void UK2Node_MyFunctionAsyncAction::AllocateDefaultPins()
{
	Super::AllocateDefaultPins();
}

bool UK2Node_MyFunctionAsyncAction::HandleDelegates(const TArray<FBaseAsyncTaskHelper::FOutputPinAndLocalVariable>& VariableOutputs, UEdGraphPin* ProxyObjectPin, UEdGraphPin*& InOutLastThenPin, UEdGraph* SourceGraph, FKismetCompilerContext& CompilerContext)
{
	bool bIsErrorFree = true;

	for (TFieldIterator<FMulticastDelegateProperty> PropertyIt(ProxyClass); PropertyIt && bIsErrorFree; ++PropertyIt)
	{
		UEdGraphPin* LastActivatedThenPin = nullptr;
		bIsErrorFree &= FBaseAsyncTaskHelper::HandleDelegateImplementation(*PropertyIt, VariableOutputs, ProxyObjectPin, InOutLastThenPin, LastActivatedThenPin, this, SourceGraph, CompilerContext);
	}

	return bIsErrorFree;
}

```

## Blueprint Effect:

On the left is the node generated by the engine's built-in UK2Node_AsyncAction, while on the right is the custom UK2Node_MyFunctionAsyncAction blueprint node. Although they have identical functions, the right side includes an additional comment for differentiation. With this foundation, you can further customize by overloading methods within it.

![Untitled](Untitled.png)

## Currently used in two places in the source code:

```cpp
UCLASS(BlueprintType, meta = (ExposedAsyncProxy = "AsyncTask", HasDedicatedAsyncNode))
class GAMEPLAYMESSAGES_API UAsyncAction_RegisterGameplayMessageReceiver : public UBlueprintAsyncActionBase
{
	UFUNCTION(BlueprintCallable, Category = Messaging, meta=(WorldContext="WorldContextObject", BlueprintInternalUseOnly="true"))
	static UAsyncAction_RegisterGameplayMessageReceiver* RegisterGameplayMessageReceiver(UObject* WorldContextObject, FEventMessageTag Channel, UScriptStruct* PayloadType, EGameplayMessageMatchType MatchType = EGameplayMessageMatchType::ExactMatch, AActor* ActorContext = nullptr);

}
//Created by UK2Node_GameplayMessageAsyncAction
void UK2Node_GameplayMessageAsyncAction::GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const
{
	//...
	UClass* NodeClass = GetClass();
	ActionRegistrar.RegisterClassFactoryActions<UAsyncAction_RegisterGameplayMessageReceiver>(FBlueprintActionDatabaseRegistrar::FMakeFuncSpawnerDelegate::CreateLambda([NodeClass](const UFunction* FactoryFunc)->UBlueprintNodeSpawner*
	{
		UBlueprintNodeSpawner* NodeSpawner = UBlueprintFunctionNodeSpawner::Create(FactoryFunc);
		check(NodeSpawner != nullptr);
		NodeSpawner->NodeClass = NodeClass;

		TWeakObjectPtr<UFunction> FunctionPtr = MakeWeakObjectPtr(const_cast<UFunction*>(FactoryFunc));
		NodeSpawner->CustomizeNodeDelegate = UBlueprintNodeSpawner::FCustomizeNodeDelegate::CreateStatic(GetMenuActions_Utils::SetNodeFunc, FunctionPtr);

		return NodeSpawner;
	}) );
}

UCLASS(BlueprintType, meta=(ExposedAsyncProxy = "AsyncTask", HasDedicatedAsyncNode))
class UMovieSceneAsyncAction_SequencePrediction : public UBlueprintAsyncActionBase
{
	UFUNCTION(BlueprintCallable, Category=Cinematics)
	static UMovieSceneAsyncAction_SequencePrediction* PredictWorldTransformAtTime(UMovieSceneSequencePlayer* Player, USceneComponent* TargetComponent, float TimeInSeconds);
}
```

## Generated Blueprint:

UAsyncAction_RegisterGameplayMessageReceiver is created by a custom UK2Node_GameplayMessageAsyncAction, providing a generic Payload output pin. UMovieSceneAsyncAction_SequencePrediction's factory method PredictWorldTransformAtTime, because the automatically generated version is hidden and no BlueprintInternalUseOnly attribute is added to suppress the UHT-generated version, results in the presentation of a standard static function blueprint node.

![Untitled](Untitled%201.png)

## The Mechanism of Action in the Source Code:

It can be observed that if HasDedicatedAsyncNode is present in a class, nullptr is returned directly, and NodeSpawner is not generated, thereby preventing the creation of blueprint nodes.

```cpp
void UK2Node_AsyncAction::GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const
{
	ActionRegistrar.RegisterClassFactoryActions<UBlueprintAsyncActionBase>(FBlueprintActionDatabaseRegistrar::FMakeFuncSpawnerDelegate::CreateLambda([NodeClass](const UFunction* FactoryFunc)->UBlueprintNodeSpawner*
	{
		UClass* FactoryClass = FactoryFunc ? FactoryFunc->GetOwnerClass() : nullptr;
		if (FactoryClass && FactoryClass->HasMetaData(TEXT("HasDedicatedAsyncNode")))
		{
			// Wants to use a more specific blueprint node to handle the async action
			return nullptr;
		}

		UBlueprintNodeSpawner* NodeSpawner = UBlueprintFunctionNodeSpawner::Create(FactoryFunc);
		check(NodeSpawner != nullptr);
		NodeSpawner->NodeClass = NodeClass;

		TWeakObjectPtr<UFunction> FunctionPtr = MakeWeakObjectPtr(const_cast<UFunction*>(FactoryFunc));
		NodeSpawner->CustomizeNodeDelegate = UBlueprintNodeSpawner::FCustomizeNodeDelegate::CreateStatic(GetMenuActions_Utils::SetNodeFunc, FunctionPtr);

		return NodeSpawner;
	}) );
}
```