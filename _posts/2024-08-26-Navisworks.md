---
title: Navisworks API Study
date: yyyy-mm-dd  +09:00
categories: [공부기록, Navisworks]
tags: [TAG]     
---
<h4> Navisworks를 사용하여 프로젝트를 빠르고 쉽게 확인하는 기능 </h4>

<h2><font color = "skyblue" > Navisworks API Study </font></h2>

<h3><font color = "hotpink"> ModelItem </font></h3> <br>
(파일 아래 항목이냐 파일 위 항목이냐)<br>
Navisworks 모델의 개별 요소(예: 객체, 구성요소).<br>
Navisworks 모델 구조의 각 항목에 대해 정보를 제공하고, 해당 항목의 속성에 접근하거나, 항목 간의 계층 구조를 탐색하는 데 사용<br>
자식,부모 항목에 접근, 속성 조회 <br>
PropertyCategory와 Property 클래스를 통해 속성 접근 
```c#
ModelItem pModelItem = pModelItem.AncestorsAndSelf  //(나 포함하여 부모 컬렉션 반환)
pModelItem.DescendantsAndSelf  //(나 포함하여 자식 컬렉션 반환)
```
<br>

**<font color = "Purple"> 주요 속성 </font><br>**
**Children** : 현재 항목의 자식 항목들<br>
**Parent**: 현재 항목의 부모 항목<br>
**PropertyCategories**: 해당 항목의 속성 카테고리<br>
**Geometry** : 모델 항목의 기하학적 데이터
**Id** : 모델 항목의 고유 식별자
**IsValid** : 모델 항목이 유효한지 여부
**Name** : 모델 항목의 이름
**Properties** : 모델 항목의 모든 속성
**Transform** : 모델 항목의 변환 행렬
**Visibility** : 모델 항목의 가시성 상태
```c#
// Children
ModelItemCollection children = modelItem.Children;

// Geometry
ModelGeometry geometry = modelItem.Geometry;

// Id
int id = modelItem.Id;

// IsValid
bool isValid = modelItem.IsValid;

// Name
string name = modelItem.Name;

// Parent
ModelItem parent = modelItem.Parent;

// PropertyCategories
PropertyCategoryCollection categories = modelItem.PropertyCategories;

// Properties
PropertyList properties = modelItem.Properties;

// Transform
Matrix transform = modelItem.Transform;

// Visibility
bool isVisible = modelItem.Visibility;
```
<br>



<h3><font color = "hotpink"> PropertyCategory  </font></h3><br>
ModelItem에 속한 속성들을 그룹으로 관리<br>
각 카테고리는 관련 속성들을 하나로 묶어 관리하며, 이를 통해 모델 항목의 다양한 속성에 효율적으로 접근할 수 있습니다. 
속성 탐색, 추가, 제거 등의 작업을 한다.
<br>

**<font color = "Purple"> 주요 속성 </font>**<br>
**카테고리 이름 및 ID**<br>
**DisplayName** : 사용자에게 표시되는 속성 카테고리의 이름. 사용자 인터페이스에서 사용되는 이름<br>
**Name** : 속성 카테고리의 내부 이름, 카테고리를 식별하는 데 사용<br>
**Id** : 속성 카테고리의 고유 식별자<br>
**CategoryType** : 속성 카테고리의 유형, 카테고리의 종류<br>
**Properties** : 속성 카테고리에 포함된 속성들의 목록<br>
**InternalName** : 내부적으로 사용되는 고유 이름. Name과 유사하지만, 더 구체적
**FindPropertyByName(string propertyName)** : 카테고리 내에서 특정 속성 이름으로 속성 검색<br>
**TryGetPropertyByName(string propertyName, out Property property)** : 지정된 이름의 속성을 검색, 속성이 존재하는지 여부를 확인하고, 결과를 out 매개변수로 반환<br>

```c#
// DisplayName
PropertyCategory category = // 얻어온 PropertyCategory 객체
string displayName = category.DisplayName;

// Name
string name = category.Name;

// Id
int id = category.Id;

// CategoryType
PropertyCategoryType categoryType = category.CategoryType;

// Properties
PropertyList properties = category.Properties;
foreach (Property prop in properties)
{
    Console.WriteLine($"Property Name: {prop.Name}, Value: {prop.Value}");
}

// InternalName
string internalName = category.InternalName;

// FindPropertyByName(string propertyName)
Property property = category.FindPropertyByName("Area");
if (property != null)
{
    Console.WriteLine($"Found Property Name: {property.Name}, Value: {property.Value}");
}

// TryGetPropertyByName(string propertyName, out Property property)
if (category.TryGetPropertyByName("Area", out Property property))
{
    Console.WriteLine($"Found Property Name: {property.Name}, Value: {property.Value}");
}
```
<br>



<h3><font color = "hotpink"> Property </font></h3><br>
ModelItem에 속한 개별 속성, 모델 항목에 대한 특정 정보를 포함한다.<br>
특정 ModelItem이 가지는 속성의 이름, 값, 그리고 관련 정보를 다룬다.<br> 
PropertyCategory 클래스의 인스턴스에 속하는 개별 속성으로, Navisworks 모델 데이터의 세부적인 정보를 포함.<br> 

**<font color = "Purple"> 주요 속성 </font><br>**
**Name** : 속성의 이름<br>
**Value** : 속성의 값을 나타내는 객체. 속성의 데이터 유형에 따라 달라진다.<br>
**IsNull** : 속성이 null인지 여부<br>
**IsReadOnly** : 속성이 읽기 전용인지 여부<br>
**DataType** : 속성 값의 데이터 유형<br>
**ToBoolean(), ToDouble(), ToString(), ToDateTime(), ToPoint3D() 등** : Value 속성의 값을 특정 데이터 타입으로 변환하는 메서드.<br>
**ToDisplayString()** : 속성 값을 사람이 읽기 쉬운 형식의 문자열로 반환.
```c#
            // Name: 속성의 내부 이름을 가져옴
            Console.WriteLine("Name: " + property.Name);

            // Value: 속성의 값을 가져옴 (VariantData 형식)
            VariantData value = property.Value;

            // IsNull : 값이 null인지 여부
            Console.WriteLine("IsNull: " + property.IsNull);

            // IsReadOnly : 읽기 전용 여부
            Console.WriteLine("IsReadOnly: " + property.IsReadOnly);

            // DataType: VariantData의 데이터 유형을 가져옴 (VariantDataType 형식)
            Console.WriteLine("DataType: " + property.DataType);
```
<br>



<h3><font color = "hotpink"> VariantData </font></h3><br>
다양한 타입의 데이터를 하나의 통일된 인터페이스로 관리할 수 있다.<br>
속성 값의 다양한 데이터 유형을 처리하기 위한 클래스.<br>
다양한 데이터 형식 (예: Int32, Double, String)을 지원.<br>
다양한 데이터 타입의 속성 값을 관리하기 위해 사용.<br>
다양한 데이터 타입을 일관되게 처리할 수 있도록 지원하며, 데이터를 적절한 형식으로 변환하거나 디스플레이할 수 있는 다양한 기능을 제공<br>

**<font color = "Purple"> 주요 속성 </font><br>**
**데이터 타입<br>**
**DataType** : 현재 저장된 데이터의 타입 <br>
VariantDataType 열거형을 반환하며, 데이터가 Boolean, Double, String, DateTime, Point3D 등 어떤 타입인지를 나타낸다.<br>
ToBoolean(), ToDouble(), ToString() 등, 데이터를 특정 형식으로 변환 <br>

**데이터 상태 관리<br>**
**IsNull** : 데이터가 null인지 여부 확인<br>
**chkNull()** : 데이터가 null이면 예외 <br>
데이터가 유효한지 확인하는 데 사용 <br>

**데이터 디스플레이<br>**
**ToDisplayString()** : 데이터를 사람이 읽기 쉬운 형식의 문자열로 반환<br>
UI에서 속성 값을 표시할 때 유용
```c#
// DataType
 // VariantData의 데이터 유형을 가져옴
    VariantDataType type = data.DataType;

    // 데이터 유형에 따라 출력
    switch (type)
    {
        case VariantDataType.Boolean:
            Console.WriteLine("DataType is Boolean");
            break;
        case VariantDataType.Double:
            Console.WriteLine("DataType is Double");
            break;
        case VariantDataType.String:
            Console.WriteLine("DataType is String");
            break;
        case VariantDataType.DateTime:
            Console.WriteLine("DataType is DateTime");
            break;
        case VariantDataType.Point3D:
            Console.WriteLine("DataType is Point3D");
            break;
        default:
            Console.WriteLine("Unknown DataType");
            break;
    }


// IsNull
    // 데이터가 null인지 확인
    if (data.IsNull)
    {
        Console.WriteLine("The data is null.");
    }
    else
    {
        Console.WriteLine("The data is not null.");
    }


// chkNull
try
    {
        // 데이터가 null인지 확인하고, null이면 예외를 발생시킴
        data.chkNull();
        Console.WriteLine("The data is valid.");
    }
    catch (Exception ex)
    {
        Console.WriteLine("The data is null. Exception: " + ex.Message);
    }
```
<br>



<h3><font color = "hotpink"> Search </font></h3><br>
Navisworks 모델에서 조건에 맞는 모델 항목을 검색하기 위해 사용<br>
특정 속성을 가진 항목이나 특정 구조적 기준에 부합하는 항목들을 필터링할 수 있다. <br>

**<font color = "Purple"> 주요 속성 </font><br>**
**SearchConditions** : 검색 조건을 관리하는 컬렉션, FindAll(), FindFirst() 등을 사용하여 조건에 맞는 모델 항목을 찾을 수 있다.<br>
**SearchBy**: 검색이 실행될 때 사용할 기준 설정, 기본적으로는 SearchBy.Children이 사용되며, 자식 항목들을 기준으로 검색, 
다른 옵션으로는 SearchBy.Descendants가 있다.<br>
**MatchingCriteria** : 검색 시 일치하는 항목을 어떻게 처리할지 설정, 
예를 들어, MatchingCriteria.All는 모든 조건을 만족해야 항목이 검색되고, MatchingCriteria.Any는 하나라도 만족하면 검색.<br>
**Run(Selection selection)** : 주어진 Selection 객체에 대해 검색, 결과는 ModelItemCollection으로 반환되며, 이는 검색된 ModelItem들의 컬렉션<br>
```c#
// Search.SearchConditions
// Search 인스턴스 생성
    Search search = new Search();

    // 검색 조건 생성: 특정 속성 이름과 값을 가진 항목을 찾기
    SearchCondition condition = SearchCondition.HasPropertyByName("LcOaNode", "LcOaNodeGuid");
    condition = condition.EqualValue(new VariantData("12345"));

    // 조건을 SearchConditions에 추가
    search.SearchConditions.Add(condition);

    // 결과를 전체 모델에서 검색
    ModelItemCollection results = search.Run(Autodesk.Navisworks.Api.Application.ActiveDocument.SelectionSets.RootItem.Selection);


// Search.SearchBy
// Search 인스턴스 생성
    Search search = new Search();

    // 검색 조건 생성
    SearchCondition condition = SearchCondition.HasPropertyByName("LcOaNode", "LcOaNodeGuid");
    condition = condition.EqualValue(new VariantData("12345"));

    // 조건 추가
    search.SearchConditions.Add(condition);

    // 검색 기준을 전체 모델로 설정
    search.SearchBy = SearchBy.AllModels;

    // 결과를 전체 모델에서 검색
    ModelItemCollection results = search.Run(Autodesk.Navisworks.Api.Application.ActiveDocument.Models);


// Search.MatchingCriteria
// Search 인스턴스 생성
    Search search = new Search();

    // 첫 번째 조건 추가
    SearchCondition condition1 = SearchCondition.HasPropertyByName("LcOaNode", "LcOaNodeGuid");
    condition1 = condition1.EqualValue(new VariantData("12345"));
    search.SearchConditions.Add(condition1);

    // 두 번째 조건 추가
    SearchCondition condition2 = SearchCondition.HasPropertyByName("LcOaNode", "LcOaSceneBaseUserName");
    condition2 = condition2.Contains("Admin");
    search.SearchConditions.Add(condition2);

    // 모든 조건이 일치해야 한다고 설정 (AND)
    search.MatchingCriteria = MatchingCriteria.All;


// Search.Run(Selection selection)
 // Search 인스턴스 생성
    Search search = new Search();

    // 조건 추가
    SearchCondition condition = SearchCondition.HasPropertyByName("LcOaNode", "LcOaNodeGuid");
    condition = condition.EqualValue(new VariantData("12345"));
    search.SearchConditions.Add(condition);

    // 검색을 선택된 항목에 대해 실행
    Selection selection = Autodesk.Navisworks.Api.Application.ActiveDocument.CurrentSelection;
    ModelItemCollection results = search.Run(selection);
```
<br>



<h3><font color = "hotpink"> SavedViewpoint </font></h3><br>
Navisworks 모델에서 특정 시점(뷰포인트)을 저장하고 관리하는 데 사용<br> 
특정 카메라 위치, 클리핑 평면, 하이라이트된 객체, 섹션 평면 등 다양한 상태 정보를 캡처할 수 있다.<br> 
Navisworks에서 특정 뷰포인트를 캡처하고, 이를 저장하여 나중에 다시 로드할 수 있는 기능을 제공<br>
모델 탐색 중 중요한 시점을 기록하고, 특정 뷰로 쉽게 이동할 수 있다.<br>
카메라 설정, 섹션 평면, 클리핑 평면, 애니메이션, 기타 디스플레이 설정과 관련된 정보를 관리하는 데 사용

**<font color = "Purple"> 주요 속성 </font><br>**
**Name** : SavedViewpoint의 이름을 나타내는 속성, 사용자가 특정 뷰포인트를 식별하는 데 사용<br> 
**Camera** : 현재 뷰포인트의 카메라 설정으르 가져오거나 설정하는 속성, 카메라의 위치, 방향, 시야각 등을 포함<br> 
**Sectioning** : 섹션 평면과 관련된 정보를 저장하고 관리하는 속성, 섹션 평면은 모델의 일부를 잘라내어 내부를 볼 수 있도록 한다.<br> 
**Appearance** : 현재 뷰포인트에서 보이는 객체들의 모양과 관련된 정보를 저장하는 속성, 하이라이트, 투명도, 색상 변경 등을 포함할 수 있다.<br> 
**ClippingPlanes** : 현재 뷰포인트에서 활성화된 클리핑 평면의 집합을 관리하는 속성, 클리핑 평면은 모델의 특정 부분을 숨길 수 있도록 한다.<br> 
**DisplayFlags** : 뷰포인트에 적용된 다양한 디스플레이 설정을 나타내는 속성, 이는 와이어프레임, 숨긴 선 표시, 그림자 등을 포함할 수 있다.<br> 
**Animations** : 현재 뷰포인트에서 사용할 수 있는 애니메이션 설정을 관리한다.<br> 
**Clone** : 뷰포인트의 복제본을 생성하는 메서드, 새로운 SavedViewpoint객체를 반환한다.<br> 
**IsLocked** : 뷰포인트가 수정 불가능한지 여부를 나타내는 속성, 잠긴 뷰포인트는 사용자가 실수로 변경하지 않도록 보호된다.<br>
 ```c#
 // Name
 SavedViewpoint viewpoint = new SavedViewpoint();
viewpoint.Name = "MyViewpoint";

// Camera
SavedViewpoint viewpoint = new SavedViewpoint();
viewpoint.Camera.Position = new Point3D(10, 20, 30);
viewpoint.Camera.LookAt = new Point3D(0, 0, 0);

// Sectioning
viewpoint.Sectioning.Planes.Add(new SectionPlane(new Point3D(0, 0, 0), new Vector3D(0, 1, 0)));

// Appearance
viewpoint.Appearance.OverrideColor(new Color(255, 0, 0));

// ClippingPlanes
viewpoint.ClippingPlanes.Add(new ClippingPlane(new Point3D(0, 0, 0), new Vector3D(1, 0, 0)));

// DisplayFlags
viewpoint.DisplayFlags.ShowGrid = true;

// Animations
viewpoint.Animations.Play();

// Clone
SavedViewpoint clonedViewpoint = viewpoint.Clone();

// IsLocked
viewpoint.IsLocked = true;


// SavedViewpoint 클래스 예시
public void CreateSavedViewpoint()
{
    // 새로운 SavedViewpoint 생성
    SavedViewpoint newViewpoint = new SavedViewpoint
    {
        Name = "New Viewpoint",
        IsLocked = false // 잠금 해제 상태로 설정
    };

    // 카메라 설정
    newViewpoint.Camera.Position = new Point3D(100, 100, 100);
    newViewpoint.Camera.LookAt = new Point3D(0, 0, 0);

    // 섹션 평면 추가
    newViewpoint.Sectioning.Planes.Add(new SectionPlane(new Point3D(0, 0, 0), new Vector3D(0, 1, 0)));

    // 뷰포인트를 현재 모델의 뷰포인트 목록에 추가
    var savedViewpoints = Autodesk.Navisworks.Api.Application.ActiveDocument.SavedViewpoints;
    savedViewpoints.AddCopy(newViewpoint);
}
```



<h3><font color = "hotpink"> Selection </font></h3><br>
선택된 항목을 관리하고 조작하는 데 사용<br>
사용자가 선택한 모델 항목(ModelItem)들을 저장하거나, 특정 기준에 따라 모델 항목을 선택하는 작업 수행<br>
모델 항목을 선택하거나 선택 해제할 수 있으며, 특정 기준에 따라 항목을 선택하거나 현재 선택된 항목을 조작<br>
Navisworks에서 모델을 탐색하고 상호작용하는 데 중요한 도구

**<font color = "Purple"> 주요 속성 </font><br>**
**Count** : Selection에 포함된 ModelItem의 개수
**IsEmpty** : 현재 선택된 항목이 비어 있는지 여부, 선택된 항목이 없으면 true
**SelectedItems** : 현재 선택된 항목(ModelItem)의 컬렉션 (읽기 전용)<br>
**Clear** : 현재 선택된 모든 항목을 제거(선택 해제)<br>
**Add(ModelItem modelItem)** :  주어진 ModelItem을 현재 선택에 추가<br>
**Remove(ModelItem modelItem)** : 주어진 ModelItem을 현재 선택에서 제거<br>
**SelectAll()** : 현재 문서의 모든 ModelItem 선택<br>
**Contains(ModelItem modelItem)** : 특정 ModelItem이 현재 선택에 포함되어 있는지 확인<br>

 ```c#
// Count
Selection selection = Autodesk.Navisworks.Api.Application.ActiveDocument.CurrentSelection;
int itemCount = selection.Count;

// IsEmpty
if (selection.IsEmpty)
{
    Console.WriteLine("No items are selected.");
}

// SelectedItems
ModelItemCollection selectedItems = selection.SelectedItems;
foreach (var item in selectedItems)
{
    Console.WriteLine(item.DisplayName);
}

// Clear()
selection.Clear();

// Add(ModelItem modelItem)
ModelItem item = /* some ModelItem */;
selection.Add(item);

// Remove(ModelItem modelItem)
selection.Remove(item);

// SelectAll()
selection.SelectAll();

// Contains(ModelItem modelItem)
bool isSelected = selection.Contains(item);
```



<h3><font color = "hotpink"> SelectionSet </font></h3><br>
사용자가 선택한 객체의 집합을 저장하고 관리<br>
ModelItem을 그룹으로 관리하는 기능<br>
여러 개의 ModelItem을 그룹화하여 저장하고, 이를 쉽게 재사용하거나 관리할 수 있다.<br>


**<font color = "Purple"> 주요 속성 </font><br>**
**DisplayName** :  electionSet의 표시 이름을 가져오거나 설정, 사용자 인터페이스에서 선택 집합을 식별하는 데 사용<br>
**Guid** : electionSet의 고유 식별자(Globally Unique Identifier), 선택 집합을 고유하게 식별하는 데 사용<br>
**ModelItems** : SelectionSet에 포함된 ModelItem들의 컬렉션, ModelItems 속성을 통해 SelectionSet에 포함된 항목들을 가져온다.<br>
**IsEmpty** : SelectionSet이 비어 있는지 확인하는 속성, ModelItem이 없다면 true<br>

**<font color = "Purple"> 주요 메서드 </font><br>**
**AddModelItems(IEnumerable<ModelItem> items)** : ModelItem 컬렉션을 SelectionSet에 추가<br>
**RemoveModelItems(IEnumerable<ModelItem> items)** : SelectionSet에서 지정된 ModelItem들을 제거<br>
**Clear()** : SelectionSet에 포함된 모든 ModelItem을 제거<br>
**FindModelItemByGuid(Guid guid)** : SelectionSet에서 특정 ModelItem을 GUID를 통해 찾는다.<br>



 ```c#
 // DisplayName
SelectionSet selectionSet = new SelectionSet();
selectionSet.DisplayName = "My Custom Selection Set";

// Guid
Guid selectionSetId = selectionSet.Guid;

// ModelItems
foreach (ModelItem item in selectionSet.ModelItems)
{
    Console.WriteLine(item.DisplayName);
}

// IsEmpty
if (selectionSet.IsEmpty)
{
    Console.WriteLine("The selection set is empty.");
}
 ```



