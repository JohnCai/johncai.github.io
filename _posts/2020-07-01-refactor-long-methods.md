---
layout: post
title: "代码重构的入门第一准则"
category: [TDD]
tags: [TDD, Refactor]
---

准备一个计时器，看需要多少分钟，才能说出下面这个函数做了哪些事？

```c#
public void work()
{
    拿出碗;
    拿出筷子;
    摆碗;
    摆筷子;
    坐下;
    拿起勺子;
    盛汤;
    喝汤;
    while (还没吃饱)
    {
        盛饭;
        夹菜;
        张口;
        咀嚼;
        吞咽;
    }
    脱鞋子;
    脱衣服;
    脱裤子;
    穿睡衣;
    穿睡裤;
    刷牙;
    洗脸;
    上床;
    关灯;
    躺下;
    while (不困)
    {
        刷手机;
    }
    放下手机;
    闭眼;
    拿出棍子;
    走到豆豆跟前;
    敲打豆豆;
}
```
读的时候，是不是一万匹神兽飘过：你特么到底要干嘛？

好的，重构一下，下面的代码呢？又需要多少分钟？

```c#
public void work()
{
    吃饭();

    睡觉();

    打豆豆();
}
```

### 为什么要重构？
面对遗留代码，一个团队花在「读代码」的时间，是「写代码」时间的 10 倍以上。

如果花 1 小时的时间重构一下代码，可以节省后续 10 小时的读代码时间，那么，重构应该成为程序员的基本良心。

### 什么时候重构？
有句话说：如果不痛，就不要重构。

不是所有代码都需要再去读，从来不需要去读的代码，就不需要重构。

反过来说，若因为加功能或者任何原因，需要读某一段代码，就应该对它进行重构。

### 如何开始重构？

都知道抽方法是最常见的重构手法之一，可是什么时候该抽方法呢？

这边给一个**我建议的重构入门第一准则：所有的 Public 方法，由且只由 Private 方法组成。**

如果上面的`吃饭();` `睡觉();` `打豆豆();` 就是 3 个私有方法，对其适当地命名，让名字清晰表达它的意图。

如此，你几乎不需要写注释。


### 实例
学习 TDD 也好，重构也好，一个很大的障碍，就是没办法把例子里学到的东西应用到生产代码中。

下面是我现实中的一个例子，比上面的傻瓜示例肯定更有挑战。

但是，使用上面的第一准则，已经足够让你开始向烂代码发起进攻了。

**重构前：**（麻烦您的鼠标了）
```vb
Public Function OutputVAT() As CN.ExportChinaVATInvoiceResponse
    Dim resp As New CN.ExportChinaVATInvoiceResponse
    Dim output As CN.ChinaVATInvoice = Nothing
    'Dim outputList As New List(Of CN.ChinaVATInvoice)
    Dim setting = New Xml.XmlWriterSettings
    Dim sbSummary As New StringBuilder
    'Dim saveChinaVATJob As SaveOutputChinaVATXMLToDisk = Nothing
    Dim v As CN.Validation = Nothing
    Dim bFileCreateSuccess As Boolean = False
    Dim logs As New List(Of EISChinaVatLog)

    Try
        Dim firstReq As CN.ExportChinaVATInvoiceRequest
        firstReq = _expRequests(0)
        'PublishExportRequest()
        resp.Validation = ValidateRequestForVATOutput()

        Dim sw1 As New IO.StringWriter
        Dim ser As New XmlSerializer(firstReq.GetType())
        ser.Serialize(sw1, firstReq)
        Dim requestString As String = sw1.ToString()
        requestString = requestString.Replace("utf-16", "utf-8")
        sw1.Close()

        Dim logSrc As New ExportChinaVATDataSource
        logSrc.InsertLogForChinaVATExport("Export China VAT Request",
                                            firstReq.GFSVATGroupNumber,
                                            requestString)


        If resp.Validation IsNot Nothing AndAlso resp.Validation.HasErrors Then
            Return resp
        End If

        setting.OmitXmlDeclaration = True
        setting.Indent = True
        setting.Encoding = New UTF8Encoding(False)

        Dim writerSummary As Xml.XmlWriter = XmlTextWriter.Create(sbSummary, setting)

        Init()

        writerSummary.WriteStartDocument()
        writerSummary.WriteStartElement("ChinaVATInvoices")

        For Each req As CN.ExportChinaVATInvoiceRequest In _expRequests
            writerSummary.WriteStartElement("ChinaVATInvoice")
            writerSummary.WriteStartElement("VATInvoiceItems")

            If req IsNot Nothing Then
                If req.CustomerCode IsNot Nothing AndAlso req.CustomerCode.Trim.Length > 0 Then
                    GetCustomerinfo(req.CustomerCode, req)
                End If

                If req.CHRBranchCode IsNot Nothing AndAlso req.CHRBranchCode.Trim.Length > 0 Then
                    GetBranchInfo(req.CHRBranchCode, req)
                End If

                For Each invRate As CN.VATInvoiceItem In req.VATInvoiceItem
                    output = ConvertToChinaVATInvoice(invRate, req)

                    If output IsNot Nothing Then
                        writerSummary.WriteStartElement("VATInvoiceItem")

                        writerSummary.WriteStartElement("VATInvoiceType")
                        writerSummary.WriteValue(output.VATInvoiceType)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("GFSVATGroupNumber")
                        writerSummary.WriteValue(output.GFSVATGroupNumber)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("VATTaxinvoiceDate")
                        If req IsNot Nothing AndAlso req.VATTaxinvoiceDate <> Date.MinValue AndAlso
                            req.VATTaxinvoiceDate.ToString <> "1/1/0001 12:00:00 AM" Then
                            writerSummary.WriteValue(String.Format("{0}-{1}-{2}", req.VATTaxinvoiceDate.Year.ToString,
                                                (req.VATTaxinvoiceDate.Month + 100).ToString.Substring(1, 2),
                                                (req.VATTaxinvoiceDate.Day + 100).ToString.Substring(1, 2)))
                        End If
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("CustomerNumber")
                        writerSummary.WriteValue(output.CustomerNumber)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("CustomerName")
                        writerSummary.WriteValue(output.CustomerName)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("CustomerTaxID")
                        writerSummary.WriteValue(output.CustomerTaxID)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("CustomerDetail")
                        writerSummary.WriteValue(String.Format("{0} {1}", output.CustomerAddress, output.CustomerTelephoneNumber))
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("CustomerBankInfo")
                        writerSummary.WriteValue(String.Format("{0} {1}", output.CustomerBankName, output.CustomerBankAccountNumber))
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("Remarks")
                        writerSummary.WriteValue(output.Remarks)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("GovernmentApprovalNumber")
                        writerSummary.WriteValue(output.GovernmentApprovalNumber)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("OriginalVATBatchID")
                        writerSummary.WriteValue(output.OriginalVATBatchID)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("OriginalVATInvoiceNumber")
                        writerSummary.WriteValue(output.OriginalVATInvoiceNumber)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("VATInvoiceBy")
                        writerSummary.WriteValue(output.VATInvoiceBy)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("VATApprovedBy")
                        writerSummary.WriteValue(output.VATApprovedBy)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("VATGroupedBy")
                        writerSummary.WriteValue(output.VATGroupedBy)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("CHRBranchBankInfo")
                        writerSummary.WriteValue(String.Format("{0} {1}", output.CHRBranchBankAccountName, output.CHRBranchBankAccountNumber))
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("CHRBranchInfo")
                        writerSummary.WriteValue(String.Format("{0} {1}", output.CHRBranchAddress, output.CHRBranchTelephoneNumber))
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("ItemNumber")
                        If output.CommodityItemNumber IsNot Nothing Then
                            writerSummary.WriteValue(output.CommodityItemNumber)
                        End If
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("ChargeName")
                        writerSummary.WriteValue(output.ChargeName)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("ModelNumber")
                        If output.ModelNumber IsNot Nothing Then
                            writerSummary.WriteValue(output.ModelNumber)
                        End If
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("ChargeBasis")
                        writerSummary.WriteValue(output.ChargeBasis)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("Quantity")
                        writerSummary.WriteValue(output.Quantity)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("Amount")
                        writerSummary.WriteValue(output.TotalInvoiceAmount)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("VATRate")
                        writerSummary.WriteValue(output.VATRate)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("VATAmount")
                        writerSummary.WriteValue(output.TaxAmount)
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("DiscountAmount")
                        If output.DiscountAmount IsNot Nothing Then
                            writerSummary.WriteValue(output.DiscountAmount)
                        End If
                        writerSummary.WriteEndElement()

                        writerSummary.WriteStartElement("DiscountTaxAmount")
                        If output.DiscountTaxAmount IsNot Nothing Then
                            writerSummary.WriteValue(output.DiscountTaxAmount)
                        End If
                        writerSummary.WriteEndElement()

                        writerSummary.WriteEndElement()

                        'SaveBatchInfo(output)
                    End If
                Next
                writerSummary.WriteEndElement()
                writerSummary.WriteEndElement()
            End If

            'Build Log if there were any warnings and user override those.
            If (req.InvoiceWarnings IsNot Nothing) Then

                For Each logReq In req.InvoiceWarnings
                    Dim invoiceNum As Long
                    If Long.TryParse(logReq.InvoiceNum, invoiceNum) Then
                        logs.Add(New EISChinaVatLog With {
                                                            .BatchID = req.BatchID,
                                                            .ChinaVatBatchID = req.ChinaVATBatchID,
                                                            .CreateDate = DateTime.Now,
                                                            .InvoiceNum = invoiceNum,
                                                            .UserId = req.SEVERLETTER,
                                                            .Operation = "WarningOverride",
                                                            .Notes = String.Join(",", logReq.Notes)})
                    End If
                Next
            End If

        Next

        writerSummary.WriteEndElement()
        writerSummary.WriteEndDocument()
        writerSummary.Flush()
        writerSummary.Close()
        'Debug.Print(sbSummary.ToString)
        Dim mediator As EDIExport.ISendChinaVATService = EDIExport.SendChinaVATFactory.GetSendChinaVATService

        mediator.SetSendParamenter(firstReq.GFSVATGroupNumber, sbSummary.ToString, firstReq.CHRBranchCode, firstReq.VATGroupedBy)

        ' Attempt to create the VAT file.
        bFileCreateSuccess = mediator.Send()

        If bFileCreateSuccess Then
            For Each log In logs
                EnterpriseInvoiceDAO.InsertEISChinaVatLog(log)
            Next

        End If

        logSrc.InsertLogForChinaVATExport("Export China VAT Resp",
                                            output.GFSVATGroupNumber,
                                            sbSummary.ToString)

    Catch ex As Exception
        ExceptionManager.Publish(ex)
    End Try

    If resp.Validation Is Nothing OrElse bFileCreateSuccess = False Then
        If resp.Validation Is Nothing Then
            resp.Validation = New CN.Validation(False, False, Nothing, Nothing)
            Dim vrList As New List(Of CN.ValidationResult)
            vrList.Add(New CN.ValidationResult With {.Text = "Failed to create VAT file."})
            resp.Validation.Errors = vrList
            resp.Validation.HasErrors = True
        Else
            If resp.Validation.Errors Is Nothing Then
                resp.Validation.Errors = New List(Of CN.ValidationResult)
                resp.Validation.Errors.Add(New CN.ValidationResult With {.Text = "Failed to create VAT file."})
                resp.Validation.HasErrors = True
            Else
                resp.Validation.Errors.Add(New CN.ValidationResult With {.Text = "Failed to create VAT file."})
                resp.Validation.HasErrors = True
            End If
        End If
    End If

    Return resp
End Function
```

**重构后：**
```vb
Public Function ExportVATInvoice() As ExportChinaVATInvoiceResponse
    LogRequest()

    Dim resp As New ExportChinaVATInvoiceResponse With {
        .Validation = ValidateRequest()
    }
    If resp.Validation.HasErrors Then
        Return resp
    End If

    Dim xml = BuildXml()

    Dim fileCreateSucceeded = TryCreateVATFile(xml)

    If fileCreateSucceeded Then
        LogWarnings()
    End If

    LogResponse(xml)

    If fileCreateSucceeded = False Then
        AddErrorsToResponse(resp)
    End If

    Return resp
End Function
```

不知道谁说的（有人说是我说的）:
>程序员看到很长的代码，应该像屎壳郎看到屎一样，兴奋地扑上去。

烂代码有个绰号，叫「屎山」。沧浪之水浊兮可以濯吾足，化身屎壳郎，为人民服务，也挺光荣的。