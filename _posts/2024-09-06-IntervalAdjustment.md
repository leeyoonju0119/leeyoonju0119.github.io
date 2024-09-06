---
title: Interval Adjustment
date: yyyy-mm-dd  +09:00
categories: [구현 기능, BIM]
tags: [TAG]     
---
# Plant BIM

<br/>

 <h3> 이동할 객체와 기준이 될 객체를 선택하여 원라는 거리까지 간격조정하는 기능</h3><br>

 ![캡처](https://github.com/user-attachments/assets/edb82205-a514-4e60-9fb0-87de12d42ea3)

 <h3><font color = "Red" > ① </font> 이동할 객체의 정보가 나온다.</h3>

  <h3><font color = "Red" > ② </font> 기준이 될 객체의 정보가 나온다.</h3>

<h3><font color = "Red" > ③ </font> 오류가 나지않는 최소길이와 최대길이가 나오며 그 범위 안에서만 이동이 가능하다. Distance에 이동할 값을 입력하고 Confirm버튼을 누르면 입력한 범위만큼 이동된다.</h3>

```c#

                    // 현재 위치 가져오기
                    LocationPoint selectLocationPoint = CurrentMoveItem.selectFamilyInstance.Location as LocationPoint;
                    XYZ selectFamilyInstanceXYZ = selectLocationPoint.Point;

                    Element element = doc.GetElement(CurrentMoveItem.selectElement);


                    if (element is Pipe)
                    {
                        Autodesk.Revit.DB.Line line = Autodesk.Revit.DB.Line.CreateBound(CurrentMoveItem.selectPipeXYZ, selectFamilyInstanceXYZ);

                        XYZ moveXYZ = CurrentMoveItem.selectPipeXYZ + line.Direction.Multiply(CurrentMoveItem.SaveDistance);

                        ElementTransformUtils.MoveElement(doc, CurrentMoveItem.selectFamilyInstance.Id, moveXYZ);

                    }

                    else
                    {
                        if (CurrentMoveItem.comparison > CurrentMoveItem.SaveDistance)
                        {
                            ElementTransformUtils.MoveElement(doc, CurrentMoveItem.selectFamilyInstance.Id, CurrentMoveItem.distanceXYZ);
                        }
                        else
                        {
                            ElementTransformUtils.MoveElement(doc, CurrentMoveItem.selectFamilyInstance.Id, -CurrentMoveItem.distanceXYZ);
                        }

                    }
```
