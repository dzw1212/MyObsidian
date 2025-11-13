# 1. 添加Replicated标识符

# 2. OnRep回调（可选）

# 3. 重写GetLifetimeReplicatedProps（必须）

```cpp
// MyActor.h
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    AMyActor();

    UPROPERTY(Replicated, BlueprintReadWrite, Category = "Stats")
    float Health;

    UPROPERTY(ReplicatedUsing = OnRep_Score, BlueprintReadOnly, Category = "Stats")
    int32 Score;

    UPROPERTY(Replicated, BlueprintReadWrite, Category = "Player")
    FString PlayerName;

    // 复制通知函数
    UFUNCTION()
    void OnRep_Score();

    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;

private:
    UFUNCTION(Server, Reliable)
    void Server_UpdateHealth(float NewHealth);
};

// MyActor.cpp
AMyActor::AMyActor()
{
    bReplicates = true;
    Health = 100.0f;
    Score = 0;
}

void AMyActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    
    // 只复制给所有客户端
    DOREPLIFETIME(AMyActor, Health);
    
    // 只复制给所有者客户端
    DOREPLIFETIME_CONDITION(AMyActor, AmmoCount, COND_OwnerOnly);
    
    // 初始复制（只复制一次）
    DOREPLIFETIME_CONDITION(AMyActor, PlayerName, COND_InitialOnly);
}

void AMyActor::OnRep_Score()
{
    // 当分数更新时，更新UI等
    OnScoreChanged.Broadcast(Score);
}

void AMyActor::Server_UpdateHealth_Implementation(float NewHealth)
{
    Health = NewHealth;  // 这个修改会自动复制到所有客户端
}
```