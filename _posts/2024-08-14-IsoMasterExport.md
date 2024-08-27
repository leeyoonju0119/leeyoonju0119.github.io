---
title: IsoMasterExpoprt
date: yyyy-mm-dd  +09:00
categories: [구현 기능, ISO]
tags: [TAG]     
---
# IsoMater

<br/>
 <h3> Autodesk Revit에서 작성된 Pipe 모델을 AutoCAD 기반 Single Line ISO-Metric 도면으로 생성하는 솔루션 </h3><br>

 <h3> 추출할 범위를 선택하는 공간 </h3>

![IsoMaster Export](https://github.com/user-attachments/assets/f608a334-4571-4922-817f-7a1fd8347f0c)

<font color = "Red" > ① </font> 3가지 방식으로 추출할 범위를 지정할 수 있다. <br>
Active View Pipeline List - 현재 View에서 보여지는 PipeLine <br>
Document All Pipelines List - 전체 도면의 PipeLine <br>
Select Elements Pipelines List - 선택한 Element의 PipeLine <br>

```c#
        private void SelectExportCheck()
        {
            if (LineListDataItems.Instance.IsActive)
            {
                // Revit 문서에서 Element를 수집
                FilteredElementCollector collector = new FilteredElementCollector(document, uidoc.ActiveView.Id);

                List<string> pipeLineNumberList = new List<string>();
                foreach (Element viewElement in collector)
                {
                    string pipeLineItem = ExportPipeLine.GetPipeLine(viewElement);

                    if (!pipeLineNumberList.Contains(pipeLineItem) && pipeLineItem != "" && pipeLineItem != null && pipeLineItem != "_")
                    {
                        pipeLineNumberList.Add(pipeLineItem);
                    }
                }


                foreach (string pipeLineNumber in pipeLineNumberList)
                {
                    LineListData data = LineListData.LineListDataValue(pipeLineNumber);

                    LineListDataItems.Instance.LineListDatas.Add(data);
                }
            }

            else if (LineListDataItems.Instance.IsAll)
            {
                ExportPipeLine.PipeAll(doc);
            }

            else
            {
                List<ElementId> elementIds = null;
                ICollection<ElementId> selectedIds = uiApp.ActiveUIDocument.Selection.GetElementIds();

                // 이미 선택된 Element가 있는지 확인
                if (selectedIds.Count > 0)
                {
                    elementIds = selectedIds.ToList();
                }
                else
                {
                    elementIds = new List<ElementId>();

                    ISelectionFilter elementSelectionFilter = new ElementSelectionFilter();
                    IList<Autodesk.Revit.DB.Reference> selectedElements = uiApp.ActiveUIDocument.Selection.PickObjects(ObjectType.Element, elementSelectionFilter, "Elements 선택하세요");
                    elementIds = SelectExportPipeLine.Select(selectedElements);
                }

                List<string> pipeLineNumberList = new List<string>();

                foreach (ElementId elementId in elementIds)
                {
                    Element element = document.GetElement(elementId);
                    string pipeLineItem = ExportPipeLine.GetPipeLine(element);

                    if (!pipeLineNumberList.Contains(pipeLineItem) && pipeLineItem != "" && pipeLineItem != null && pipeLineItem != "-")
                    {
                        pipeLineNumberList.Add(pipeLineItem);
                    }
                }

                foreach (string pipeLineNumber in pipeLineNumberList)
                {
                    LineListData data = LineListData.LineListDataValue(pipeLineNumber);

                    LineListDataItems.Instance.LineListDatas.Add(data);
                }
            }
        }
```

<br>
<br>
<br>
<br>
<br>

 <h3> 저장할 위치, 추출할 PipeLine을 선택하여 추출하고 실시간으로 결과를 볼 수 있는 공간 </h3>

![IsoMaster Export LineList](https://github.com/user-attachments/assets/2538bd14-6c46-43f4-bf73-bb9237af136b)

<font color = "Red" > ① </font> 추출한 PCF파일을 저장할 위치를 지정한다. <br>
"폴더선택" 버튼을 누르면 폴더를 선택할 수 있는 창이 열리고 선택이 가능하다. <br>

<font color = "Red" > ② </font> Select All 체크하면 전체가 선택되고 상단의 Text Box로 필터기능도 가능하며 총 PipeLine 개수와 선택된 PipeLine 개수를 확인할 수 있다. <br> 
출력됨과 동시에 Result의 결과를 확인할 수 있다

<font color = "Red" > ③ </font> 출력과 동시에 에러, 경고인 PipeLine이 실시간으로 표시되고 해당 PipeLine, ElementId, 출력 결과, 이유가 나오고 더블클릭시 해당 문제가 있는 ElementId로 이동하여 이유를 알 수 있어 바로 수정이 가능하다.

<font color = "Red" > ④ </font> Export ISO 버튼을 누르면 출력이 시작되고 프로그래스바로 현 상황이 파악 가능하다.

저장할 파일을 사용자가 선택할 수 있게 문서찯으로 폴더 선택한다.

```c#
        private void BtnFolder_Click(object sender, RoutedEventArgs e)
        {
            FolderBrowserDialog folderBrowserDialog = new FolderBrowserDialog();

            // 다이얼로그에 표시되는 설명
            folderBrowserDialog.Description = "Excel 파일을 저장할 폴더를 선택하세요.";

            // 대화 상자가 표시되고 사용자가 "OK"를 클릭했을 때만 실행
            if (folderBrowserDialog.ShowDialog() == System.Windows.Forms.DialogResult.OK)
            {
                // 선택된 폴더의 경로를 TextBox에 표시
                TbxResultFolder.Text = folderBrowserDialog.SelectedPath;
            }
        }

```

필터기능으로 원하는 PipeLine을 검색하여 추출
```c#
            string searchText = TbxSearch.Text.ToUpper();

            // 입력된 문자열이 없으면 모든 데이터를 표시
            if (string.IsNullOrEmpty(searchText))
            {
                LvwLineList.ItemsSource = LineListDataItems.Instance.LineListDatas;
                return;
            }

            /// 입력된 문자열을 포함하는 데이터만 필터링하여 리스트뷰에 표시
            var filteredData = LineListDataItems.Instance.LineListDatas.Where(item => item.PipeLineItem.Contains(searchText)).ToList();
            LvwLineList.ItemsSource = filteredData;
```

Error List에 정보 입력하여 내보내기
```c#
        public void ResultError(LineListData exportItem, string pipeLineName, ElementId elementId)
        {
                exportItem.Result = "Error";
                ErrorLine = pipeLineName;
                Result = "Error";
                ElementId = elementId;

            if (ErrorListDataItem.Instance.DifferentPipeline)
            {
                Reason = ErrorAbbreviation.Instance.GetErrorReason("Different Connected Pipeline");
            }
            else
            {
                Reason = ErrorAbbreviation.Instance.GetErrorReason(ErrorListDataItem.ErrorKey);
            }
        }
```

Line List에 결과 입력하여 내보내기
```c#
        public void ResultSuccess(LineListData exportItem, string pipeLineName, ElementId elementId)
        {
            exportItem.Result = "Success";

            if (ErrorListDataItem.IsWarning)
            {
                exportItem.Result = "Warning";

                ErrorLine = pipeLineName;
                Result = "Warning";
                ElementId = elementId;

                Reason = ErrorAbbreviation.Instance.GetErrorReason(ErrorListDataItem.WarningKey);
            }
        }
```

<br>
<br>
<br>
<br>
<br>

 <h3> Progress Bar</h3>

![image](https://github.com/user-attachments/assets/0904390d-739a-43ff-9a97-e504bbb7b84e)

PipeLine Name에는 현재 출력중인 PipeLine이 나오고 현재 진행중인 개수 / 총 개수, 하나의 PipeLine 진행률, 전체 출력 진행률이 나와 파악하기 쉽다. <br>
 Stop을 누르면 일시정지되며 다시 전 화면으로 돌아간다.

<br>
<br>
<br>
<br>
<br>

![export완](https://github.com/user-attachments/assets/dadc7c6f-3977-4db7-b548-bfcfd6d25055)

출력이 완료되면 출력에 성공한 개수와 실패한 개수가 나오고 Line List의 결과와 Error List의 Error, Warning이 나온다 