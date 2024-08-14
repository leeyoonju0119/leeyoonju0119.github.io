---
title: IsoMasterOption
date: yyyy-mm-dd  +09:00
categories: [내가 만든 기능, ISO]
tags: [TAG]     
---
# IsoMater

<br/>
 <h3> Autodesk Revit에서 작성된 Pipe 모델을 AutoCAD 기반 Single Line ISO-Metric 도면으로 생성하는 솔루션 </h3><br>
 <font color = "Red" > PMS Data Mapping  </font>은 Excel형식의 미리 정의한 매핑테이블과 연동되는 공간

![Isomaster Option PMS Data Mapping](https://github.com/user-attachments/assets/48ead1ef-3e0a-4e57-816d-e1ff02400bcf)
<br>
<font color = "Red" > **①** </font>은 파일 불러오기로 ...버튼을 클릭시 파일을 선택할 수 있는 창이 열어지고 선택이 가능하다.<br> 
OpenFileDialog로 필터써서 구현. <br>
```c#
OpenFileDialog saveFileDialog = OpenFileDialog();
saveFileDialog.Filter = "Excel 파일 (*.xlsx)|*.xlsx|모든 파일 (*.*)|*.*"; 
```

<font color = "Red" > **②** </font> Parameter Location콤보박스는 Type, Instance, Both(Type First), Both(Instance First)로 구성되어 있고 <br>
Parameter Name은 Revit에서 Element를 수집하고 문서의 ParameterBindings을 가져온다<br>
사용자 정의 파라미터만 추가하기 위해 HashSet으로 BuiltInParameter를 관리한다 <br>
ParameterElement 반복문 돌려서 TypeBinding인지 InstanceBinding인지 조건문 걸어서 맞는 리스트에 저장한다.
```c#
            // Revit 문서에서 Element를 수집
            FilteredElementCollector collector = new FilteredElementCollector(document)
                .OfClass(typeof(ParameterElement));

            // 문서의 ParameterBindings 가져오기
            BindingMap bindingMap = document.ParameterBindings;

            // 사용자 정의 파라미터만 추가하기 위해 HashSet으로 BuiltInParameter를 관리
            HashSet<ElementId> builtInParameterIds = new HashSet<ElementId>(
                Enum.GetValues(typeof(BuiltInParameter))
                .Cast<BuiltInParameter>()
                .Select(bip => new ElementId(bip))
                );
```

ExcelHeader는 매핑테이블과 매칭될 Header이름을 적어 매핑테이블에서 맞는 Header을 찾는다.

<font color = "Red" > **③** </font>도 마찬가지로 매핑테이블과 매칭될 From Size와 To Size의 Excel Header이름을 넣는다.<br>

<font color = "Red" > **④** </font>는 Apply는 창이 닫히지 않고 저장, Save는 저장 후 닫기, Close는 닫기<br>

