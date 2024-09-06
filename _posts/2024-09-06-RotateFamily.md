---
title: RotateFamily
date: yyyy-mm-dd  +09:00
categories: [구현 기능, BIM]
tags: [TAG]     
---
# Plant BIM

<br/>
 <h3> 선택한 Family를 회전하는 기능</h3><br>

![캡처](https://github.com/user-attachments/assets/30efd544-62a6-45e6-9e0a-073b61cd58ef)

<br>
<font color = "Red" > ① </font> 회전할 Family를 선택한다 (필터기능으로 선택한 부재만 선택이 가능하도록).<br>

<font color = "Red" > ② </font> 선택한 객체의 정보들이 나온다.<br>

<font color = "Red" > ③ </font> 모델링한 순서대로 방향이 정해져있어 시계방향, 반시계반향으로 얼만큼 회전할건지 적고 Confilm을 누르면 회전된다.<br>

```c#
            Autodesk.Revit.DB.Line valveLine1 = Autodesk.Revit.DB.Line.CreateBound(valveStXYZ + new XYZ(1, 1, 1), valveEdXYZ + new XYZ(1, 1, 1));

            if (valveItems.pipeLines.Direction.ToString() == valveLine1.Direction.ToString())
            {
                Autodesk.Revit.DB.Line valveLine2 = Autodesk.Revit.DB.Line.CreateBound(valveStXYZ, valveEdXYZ);

                if (CurrentValveItems.isRightChecked)
                {
                    double rotationAngle = ValvesItem.SaveDeg * (-Math.PI / 180.0);
                    ElementTransformUtils.RotateElement(doc, valve.Id, valveLine2, rotationAngle);
                }
                else if (CurrentValveItems.isLeftChecked)
                {
                    double rotationAngle = ValvesItem.SaveDeg * (Math.PI / 180.0);
                    ElementTransformUtils.RotateElement(doc, valve.Id, valveLine2, rotationAngle);
                }
            }
```
라인을 그려주고 그 라인과 방향이 맞으면 Deg에 적힌 값만큼 회전한다.