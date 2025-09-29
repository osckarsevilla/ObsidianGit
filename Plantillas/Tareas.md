---
tags:
  - task
  - tareas
type: Listado
descripción: Tareas pendientes, en revisión o ya realizadas
fecha: <%tp.date.now("DD/MM/YYY")%>
fecha ultima modificación: <% tp.file.last_modified_date("dddd Do MMMM YYYY HH:mm") %>
nombre: <%tp.file.title%>
estados:
  - realizada
  - pendientes
  - sin prisa
  - urgente
banner: task
banner_y: "40"
banner-fade: "-500"
content-start: "50"
Fecha creación:
---
# [[<%tp.file.title%>]]

<%*await tp.file.move("/Tareas/" + tp.file.title)%>

 Fecha de Inicio: <%tp.date.now("DD-MM-YYYY")%>


