---
layout: post
title: "程序员的基本良心"
category: [TDD]
tags: [TDD, Refactor]
---

弄一个计时工具，看看你需要花多少分钟，才能明确地说出来下面的代码做了什么事情？
```VBA
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

重置一下计时器，如果要说出来下面的代码做了什么事，又需要多少分钟？
```VB
        Public Function OutputVAT() As CN.ExportChinaVATInvoiceResponse
            LogRequest()

            Dim resp As New ExportChinaVATInvoiceResponse With {
                .Validation = ValidateRequestForVATOutput()
            }
            If resp.Validation.HasErrors Then
                Return resp
            End If

            Dim xml As StringBuilder = BuildXml()

            Dim fileCreateSuccess = TryCreateVATFile(xml)

            If fileCreateSuccess Then
                LogWarnings()
            End If

            LogResponse(xml)

            If fileCreateSuccess = False Then
                AddErrorsToResponse(resp)
            End If

            Return resp
        End Function
```

有什么感想呢？

```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```