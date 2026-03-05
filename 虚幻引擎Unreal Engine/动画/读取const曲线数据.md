虽然曲线看起来是矩形的，但如果直接去读，还是会在边界点处读到非0和1的小数；
需要将读取时插值类型设置为`EAnimInterpolationType::Step`；

```cpp
int FrameNum = AnimSequence->GetNumberOfSampledKeys();
for (int FrameIdx = 0; FrameIdx < FrameNum; ++FrameIdx)
{
	float FrameTime = FrameIdx * (1.f / 30.f);
	
	FAnimExtractContext ExtraceCurveValContext;
	ExtraceCurveValContext.CurrentTime = FrameTime;
	ExtraceCurveValContext.InterpolationOverride = EAnimInterpolationType::Step;
	
	float fCurveVal = AnimSequence->EvaluateCurveData("AttackPhase", ExtraceCurveValContext, true);
}
```

