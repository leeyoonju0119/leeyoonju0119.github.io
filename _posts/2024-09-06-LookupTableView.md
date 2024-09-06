---
title: LookupTable
date: yyyy-mm-dd  +09:00
categories: [구현 기능, BIM]
tags: [TAG]     
---
# Plant BIM

<br/>
 <h3> 선택한 아이템의 객체 정보를 편리하게 볼 수 있는 기능</h3><br>
 <h3><font color = "Red" > LookupTableView  </font> - 선택한 아이템의 객체 정보가 나타나는 화면</h3>

![그림1](https://github.com/user-attachments/assets/d60307ac-ec43-4e6f-9a43-f30651d30700)
<br>
<font color = "Red" > ① </font> Serch버튼을 누르면 revit화면에서 객체를 선택할 수 있고 선택한 객체의 모델 전체 정보를 불러온다.<br>

<br>
<font color = "Red" > ② </font> 선택한 객체에 대한 정보가 나온다. <br>

<br>
<font color = "Red" > ③ </font> 선택한 객체 모델의 전체 정보가 나온다. <br>

```c#
// 선택된 객체가 파이프인 경우
Pipe tmpPipe = ElementLookupSizeDatas.Instance.selectElement as Pipe;
PipeSegment tmpSeg = tmpPipe.PipeSegment;
ICollection<MEPSize> mepsizes = tmpSeg.GetSizes();
ElementLookupSizeDatas.Instance.Pipe = tmpPipe;

----------------------------------------------------------------------------------

// 선택된 객체가 FamilyInstance인 경우
FamilyInstance fi = ElementLookupSizeDatas.Instance.selectElement as FamilyInstance;
Family fm = fi.Symbol.Family;

// 해당 Family의 FamilySizeTableManager를 검색
FamilySizeTableManager asd = FamilySizeTableManager.GetFamilySizeTableManager(doc, fm.Id);
FamilySizeTable fst = asd.GetSizeTable(asd.GetAllSizeTableNames().FirstOrDefault());

// 테이블 크기 차원 검색
int rangeCol = fst.NumberOfColumns;
int rangeRow = fst.NumberOfRows;

// 헤더 데이터 및 테이블 데이터를 위한 리스트 초기화
List<string> headerDatas = new List<string>();
string[,] tableArray = new string[rangeRow, rangeCol];

// 헤더 데이터 채우기
for (int i = 0; rangeCol > i; i++)
{
    if (string.IsNullOrEmpty(fst.GetColumnHeader(i).Name))
    {
        headerDatas.Add("Base Size");
    }
    else
    {
        headerDatas.Add(fst.GetColumnHeader(i).Name);
    }
}

// 테이블 데이터 채우기
for (int row = 0; row < rangeRow; row++)
{
    for (int col = 0; col < rangeCol; col++)
    {
        tableArray[row, col] = fst.AsValueString(row, col);
    }
}

// FamilyInstanceLookupSize 객체 생성 및 데이터 설정
FamilyInstanceLookupSize eleData = new FamilyInstanceLookupSize();
eleData.sizeDataTables = tableArray;
eleData.headerDatas = headerDatas;
eleData.ElementData = ElementLookupSizeDatas.Instance.selectElement;

---------------------------------------------------------------------------------------------

    List<PipeSizeLookupTable> pipeSizeLookupTableList = new List<PipeSizeLookupTable>();

    foreach (MEPSize size in mepsizes)
    {
        double convertedID = ConvertSizesToInch(size.InnerDiameter);
        double convertedOD = ConvertSizesToInch(size.OuterDiameter);
        double convertedNominal = ConvertSizesToInch(size.NominalDiameter);

        bool usedInSizeLists = size.UsedInSizeLists;
        bool usedInSizing = size.UsedInSizing;
        
        Specs tmpSepc = UtilesProcesscs.FindDataByDB(tmpPipe);


        string des = tmpSepc.Description;
        string refs = tmpSepc.Reference;

        PipeSizeLookupTable pipeSizeLookupTable = new PipeSizeLookupTable
        {
            PipeData = tmpPipe,
            Nominal = ConvertToFraction(convertedNominal),
            OutsideDiameter = ConvertToFraction(convertedOD),
            InsideDiameter = ConvertToFraction(convertedID),
            UsedInSizeLists = usedInSizeLists,
            UsedInSizing = usedInSizing
        };

        pipeSizeLookupTableList.Add(pipeSizeLookupTable);
    }
    return pipeSizeLookupTableList;
}

```
<br>
<br>
<br>
<br>
<br>


![캡처](https://github.com/user-attachments/assets/0c8b0e83-5c4f-43e6-b9d4-57ee59e68b1d)

<font color = "Red" > ④ </font> 정보를 Excel파일로 저장하여 내보낼 수 있다. <br>

<br>
<br>
<br>
<br>
<br>

![캡처](https://github.com/user-attachments/assets/50480371-9363-4665-9fe8-a68981aa13a1)

출력된 Excel화면으로 기능을 실행하지 않아도 정보들을 확인할 수 있다.