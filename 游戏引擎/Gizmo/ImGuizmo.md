*https://github.com/CedricGuillemet/ImGuizmo

ImGuizmo是基于ImGui的小型库，提供可视化改变transform矩阵的能力；

# API文档

在`ImGui_XXXX_NewFrame()`之后立即调用：
```c++
void BeginFrame();
```

如果鼠标位于某个gizmo控件上（坐标轴、平面或屏幕组件），则返回`true`；
```c++
bool IsOver();
```

如果gizmo控件处于`IsOver`状态或者移动状态，则返回`true`；
```c++
bool IsUsing();
```

enable或者disable控件，若disable状态则为半透明灰色；
```c++
void Enable(bool enable);
```

transform矩阵与translation、rotation、scale组件之间的转换；
```c++
 float matrixTranslation[3], matrixRotation[3], matrixScale[3];
 ImGuizmo::DecomposeMatrixToComponents(gizmoMatrix.m16, matrixTranslation, matrixRotation, matrixScale);
 ImGui::InputFloat3("Tr", matrixTranslation, 3);
 ImGui::InputFloat3("Rt", matrixRotation, 3);
 ImGui::InputFloat3("Sc", matrixScale, 3);
 ImGuizmo::RecomposeMatrixFromComponents(matrixTranslation, matrixRotation, matrixScale, gizmoMatrix.m16);
```

渲染一个立方体，常用于debug：
```c++
void DrawCube(const float *view, const float *projection, float *matrix);
```

绘制Gizmo控件：
```c++
enum OPERATION
{
	TRANSLATE,
	ROTATE,
	SCALE
};

enum MODE
{
	LOCAL,
	WORLD
};

void Manipulate(const float *view, const float *projection, OPERATION operation, MODE mode, float *matrix, float *deltaMatrix = 0, float *snap = 0);
```

# 示例

![ImGuizmo|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230301230606.png)

```c++
void EditTransform(const Camera& camera, matrix_t& matrix)
{
	static ImGuizmo::OPERATION mCurrentGizmoOperation(ImGuizmo::ROTATE);
	static ImGuizmo::MODE mCurrentGizmoMode(ImGuizmo::WORLD);
	if (ImGui::IsKeyPressed(KWY_Z))
		mCurrentGizmoOperation = ImGuizmo::TRANSLATE;
	if (ImGui::IsKeyPressed(KEY_E))
		mCurrentGizmoOperation = ImGuizmo::ROTATE;
	if (ImGui::IsKeyPressed(KEY_R))
		mCurrentGizmoOperation = ImGuizmo::SCALE;
	if (ImGui::RadioButton("Translate", mCurrentGizmoOperation == ImGuizmo::TRANSLATE))
		mCurrentGizmoOperation = ImGuizmo::TRANSLATE;
	ImGui::SameLine();
	if (ImGui::RadioButton("Rotate", mCurrentGizmoOperation == ImGuizmo::ROTATE))
		mCurrentGizmoOperation = ImGuizmo::ROTATE;
	ImGui::SameLine();
	if (ImGui::RadioButton("Scale", mCurrentGizmoOperation == ImGuizmo::SCALE))
		mCurrentGizmoOperation = ImGuizmo::SCALE;
	float matrixTranslation[3], matrixRotation[3], matrixScale[3];
	ImGuizmo::DecomposeMatrixToComponents(matrix.m16, matrixTranslation, matrixRotation, matrixScale);
	ImGui::InputFloat3("Tr", matrixTranslation, 3);
	ImGui::InputFloat3("Rt", matrixRotation, 3);
	ImGui::InputFloat3("Sc", matrixScale, 3);
	ImGuizmo::RecomposeMatrixFromComponents(matrixTranslation, matrixRotation, matrixScale, matrix.m16);

	if (mCurrentGizmoOperation != ImGuizmo::SCALE)
	{
		if (ImGui::RadioButton("Local", mCurrentGizmoMode == ImGuizmo::LOCAL))
			mCurrentGizmoMode = ImGuizmo::LOCAL;
		ImGui::SameLine();
		if (ImGui::RadioButton("World", mCurrentGizmoMode == ImGuizmo::WORLD))
			mCurrentGizmoMode = ImGuizmo::WORLD;
	}
	static bool useSnap(false);
	if (ImGui::IsKeyPressed(KEY_S))
		useSnap = !useSnap;
	ImGui::Checkbox("", &useSnap);
	ImGui::SameLine();
	vec_t snap;
	switch (mCurrentGizmoOperation)
	{
	case ImGuizmo::TRANSLATE:
		snap = config.mSnapTranslation;
		ImGui::InputFloat3("Snap", &snap.x);
		break;
	case ImGuizmo::ROTATE:
		snap = config.mSnapRotation;
		ImGui::InputFloat("Angle Snap", &snap.x);
		break;
	case ImGuizmo::SCALE:
		snap = config.mSnapScale;
		ImGui::InputFloat("Scale Snap", &snap.x);
		break;
	}
	ImGuiIO& io = ImGui::GetIO();
	ImGuizmo::SetRect(0, 0, io.DisplaySize.x, io.DisplaySize.y);
	ImGuizmo::Manipulate(camera.mView.m16, camera.mProjection.m16, mCurrentGizmoOperation, mCurrentGizmoMode, matrix.m16, NULL, useSnap ? &snap.x : NULL);
}
```