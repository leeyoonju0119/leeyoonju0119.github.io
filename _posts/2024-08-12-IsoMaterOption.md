---
title: IsoMasterOption
date: yyyy-mm-dd  +09:00
categories: [내가 만든 기능, ISO]
tags: [TAG]     
---
# IsoMater

<br/>
 <h3> Autodesk Revit에서 작성된 Pipe 모델을 AutoCAD 기반 Single Line ISO-Metric 도면으로 생성하는 솔루션 </h3><br>
 <h3><font color = "Red" > PMS Data Mapping  </font> - Excel형식의 미리 정의한 매핑테이블과 연동되는 공간 </h3>

![Isomaster Option PMS Data Mapping](https://github.com/user-attachments/assets/48ead1ef-3e0a-4e57-816d-e1ff02400bcf)
<br>
<font color = "Red" > ① </font> 파일 불러오기로 ...버튼을 클릭시 파일을 선택할 수 있는 창이 열어지고 선택이 가능하다.<br> 
OpenFileDialog로 필터써서 구현. <br>
```c#
OpenFileDialog saveFileDialog = OpenFileDialog();
saveFileDialog.Filter = "Excel 파일 (*.xlsx)|*.xlsx|모든 파일 (*.*)|*.*"; 
```

<font color = "Red" > ② </font> Parameter Location콤보박스는 Type, Instance, Both(Type First), Both(Instance First)로 구성되어 있고 <br>
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

<font color = "Red" > **③** </font> 마찬가지로 매핑테이블과 매칭될 From Size와 To Size의 Excel Header이름을 넣는다.<br>

<font color = "Red" > **④** </font> Apply는 창이 닫히지 않고 저장, Save는 저장 후 닫기, Close는 닫기<br>
Option에서 설정한 데이터들을 XML형식으로 저장하고 다시 Option, Export를 실행할때 저장된 XML을 불러와서 데이터들을 해당하는 변수에 넣어서 사용한다. <br>
저장하기전 선택하지 않은 ComboBox가 있는지, Header값이 있는지 검사한다 <br>
PCF파일을 저장하는 위치는 "C:\Users\lyj0119\AppData\Roaming\MegaBM\IsoMaster2024"로 아래와 같이 구현했다.
```c#
            string appDataPath = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
            string folderPath = Path.Combine(appDataPath, "MegaBM", "IsoMaster2024");
            string filePath = Path.Combine(folderPath, document.Title);
```

폴더가 존재하지 않으면 폴더를 생성하여 저장할 수 있도록 구현하였다.
```c#
 try
 {
     // 폴더가 존재하지 않으면 모든 상위 폴더를 포함하여 생성
     if (!Directory.Exists(folderPath))
     {
         Directory.CreateDirectory(folderPath); 
         Console.WriteLine($"Created directory: {folderPath}");
     }
 }
 catch (Exception ex)
 {
     // 디렉토리 생성 오류 처리
     Console.WriteLine($"Error creating directory: {ex.Message}");
 }
 ```

XML 파일로 저장하는 기능을 수행하기 위해 XML 직렬화 및 파일 저장을 간단하게 처리하도록 구성했다.
```c#
    // XmlSerializer를 이용하여 객체를 XML로 직렬화
    XmlSerializer serializer = new XmlSerializer(typeof(XMLOptionSettings));

    // StreamWriter를 사용하여 XML을 파일에 저장
    using (TextWriter writer = new StreamWriter(filePath))
    {
        // isoMaster 객체를 XML로 변환하여 파일에 쓰기
        serializer.Serialize(writer, isoMaster);
    }
 ```

<br>
<br>
<br>
<br>
<br>

<h3> <font color = "Red" > PipeLine Number </font> - Revit Element PipeLine 번호와 맞도록 PipeLine 조합하는 공간 </h3>

![IsoMaster Option PipeLine Number](https://github.com/user-attachments/assets/67150b0b-cc41-45ce-a7cc-c26f269911d0)

<font color = "Red" > ① </font> No.는 조합될 순서이고 Parameter Name으로 조합할 파라미터를 선택하여 조합하여야 할 파라미터가 두가지 이상인 경우 Delimiter에 구분 지을 문자를 넣어준다(Revit PipeLine네 맞춰 없으면 생략가능)


<font color = "Red" > ② </font> New버튼은 생성으로 클릭시 자동으로 행이 추가된다. <br>
Del버튼은 Delete로 삭제할 행을 선택한 뒤 Del버튼을 누르면 해당 행이 삭제된다. <br>
Up, Down버튼은 순서롤 조절하는 버튼으로 이동할 행을 선택한 뒤 Up버튼을 누르면 위로, Down버튼을 누르면 아래로 이동한다.

New버튼 클릭 시 기존 항목 갯수를 불러와 1을 더해주어 No.를 지정하고 기존에 있던 PipeLineNoData List에 기본값으로 추가하여 넣어줬다.
```c#
            int noCount = LvwPipeLineNumber.Items.Count;

            pipeLineNoData.AddListView(noCount);

            LvwPipeLineNumber.ItemsSource = PipeLineNoDataItem.Instance.PipeLineNumberData;
 ```

```c#
                public void AddListView(int noCount)
        {
            int nextNoItem = 1;
             PipeLineNoDataItem.Instance.ParameterLocationItems.Clear();

            if (noCount == 0)
            {
                nextNoItem = 1;
            }
            else
            {
                nextNoItem = noCount + 1;
            }

            if (PipeLineNoDataItem.Instance.ParameterLocationItems.Count > 0)
            {
                PipeLineNoDataItem.Instance.ParameterLocationItems.Clear();
            }
            PipeLineNoDataItem.Instance.ParameterLocationItems.Add("Type");
            PipeLineNoDataItem.Instance.ParameterLocationItems.Add("Instance");

            var data = new PipeLineNoData
            {
                NoItem = nextNoItem,
                ParameterNameItems = DataMappingDataItem.Instance.TypeParameterNames,
                ParameterLocationItems = PipeLineNoDataItem.Instance.ParameterLocationItems,
                SelectedParameterLocationItem = PipeLineNoDataItem.Instance.ParameterLocationItems[0],
                DelimiterItem = "",
                TagParameterLocation = nextNoItem - 1
            };

            PipeLineNoDataItem.Instance.PipeLineNumberData.Add(data);
        }
 ```

<br>
<br>
<br>

Del버튼 클릭 시 선택된 항목이 없으면 동작하지 않고 선택된 각 항목을 개별적으로 삭제한다.

```c#
 // 선택된 항목이 없는 경우 리턴
 if (LvwPipeLineNumber.SelectedItems.Count == 0)
 {
     return;
 }

 // 선택된 각 항목을 개별적으로 삭제
 foreach (var selectedItem in LvwPipeLineNumber.SelectedItems.Cast<PipeLineNoData>().ToList())
 {
     PipeLineNoDataItem.Instance.PipeLineNumberData.Remove(selectedItem);
 }

 int noItem = 1;

 foreach (var pipeLineNoItem in PipeLineNoDataItem.Instance.PipeLineNumberData)
 {
     pipeLineNoItem.NoItem = noItem++;
 }
 LvwPipeLineNumber.ItemsSource = null;

 LvwPipeLineNumber.ItemsSource = PipeLineNoDataItem.Instance.PipeLineNumberData;
 ```

<br>
<br>
<br>

Up버튼 클릭 시 선택한 항목의 위치를 이전 항목과 교환하고 No.를 다시 지정한다.

```c#
if (LvwPipeLineNumber.SelectedItems.Count != null)
{
    PipeLineNoDataItem.Instance.IsUpDown = true;

    var selectedItem = LvwPipeLineNumber.SelectedItem;
    int selectedIndex = LvwPipeLineNumber.SelectedIndex;

    if (selectedIndex > 0)
    {
        var previousIndex = selectedIndex - 1;

        // 선택된 항목과 그 이전 항목의 위치를 교환
        var temp = PipeLineNoDataItem.Instance.PipeLineNumberData[selectedIndex];
        PipeLineNoDataItem.Instance.PipeLineNumberData[selectedIndex] = PipeLineNoDataItem.Instance.PipeLineNumberData[previousIndex];
        PipeLineNoDataItem.Instance.PipeLineNumberData[previousIndex] = temp;

        // 선택된 항목을 이전 항목 위로 이동
        LvwPipeLineNumber.SelectedItem = selectedItem;

        int noItem = 1;

        foreach (var pipeLineNoItem in PipeLineNoDataItem.Instance.PipeLineNumberData)
        {
            pipeLineNoItem.NoItem = noItem++;
            pipeLineNoItem.TagParameterLocation = noItem - 2;
        }

        LvwPipeLineNumber.Items.Refresh(); // ListView 갱신
    }
}
PipeLineNoDataItem.Instance.IsUpDown = false;
 ```

<br>
<br>
<br>

Down버튼 클릭 시 선택한 항목의 위치를 다음 항목과 교환하고 No.를 다시 지정한다.

```c#
            if (LvwPipeLineNumber.SelectedItems.Count == 1)
            {
                var selectedItem = LvwPipeLineNumber.SelectedItem;
                int selectedIndex = LvwPipeLineNumber.SelectedIndex;

                if (selectedIndex < LvwPipeLineNumber.Items.Count - 1)
                {
                    var nextIndex = selectedIndex + 1;

                    PipeLineNoDataItem.Instance.IsUpDown = true;

                    // 선택된 항목과 그 다음 항목의 위치를 교환
                    var temp = PipeLineNoDataItem.Instance.PipeLineNumberData[selectedIndex];
                    PipeLineNoDataItem.Instance.PipeLineNumberData[selectedIndex] = PipeLineNoDataItem.Instance.PipeLineNumberData[nextIndex];
                    PipeLineNoDataItem.Instance.PipeLineNumberData[nextIndex] = temp;

                    // 선택된 항목을 다음 항목 아래로 이동
                    LvwPipeLineNumber.SelectedItem = selectedItem;

                    int noItem = 1;

                    foreach (var pipeLineNoItem in PipeLineNoDataItem.Instance.PipeLineNumberData)
                    {
                        pipeLineNoItem.NoItem = noItem++;
                        pipeLineNoItem.TagParameterLocation = noItem - 2;
                    }

                    LvwPipeLineNumber.Items.Refresh(); // ListView 갱신

                }
            }
            PipeLineNoDataItem.Instance.IsUpDown = false;
```

<br>
<br>
<br>
<br>
<br>

 <h3><font color = "Red" > Coordinates </font>는 단위 설정하는 공간 </h3>

![IsoMaster Option Coordinates Units](https://github.com/user-attachments/assets/5b6992ff-6efd-46c7-a1a4-9c27d14e21b2)

<font color = "Red" > ① </font> nominal sizes(bore) - Element Size(직경) <br>
coordinates(length) - 길이 측정 단위 (MM, INCH)<br>
bolt diameters - 볼트 지름 (MM, INCH)<br>
bolt lengths - 볼트 길이 (MM, INCH)<br>
component weight - 구성 요소 무게 (KGS, LBS)<br>

<br>
<br>
<br>
<br>
<br>

 <h3><font color = "Red" > Options </font>는 Connector 연결 오차범위와 Tag Parameter를 설정하는 공간 </h3>

![IsoMaster Option Options](https://github.com/user-attachments/assets/68121840-fe2e-429f-be16-d4efb4b582e5)


<font color = "Red" > ① </font> Between connector는 연결이 끊긴 커넥터의 오차 범위를 설정할 수 있다. 
오차 범위안에 다른 커넥터가 있으면 오류없이 PCF출력이 가능하다 <br>



<font color = "Red" > ② </font> Special Item, Inline Instrument등 Revit에서 사용자 정의 Parameter를 불러와서 해당하는 Tag Parameter를 설정하여 추출할 수 있다<br>

