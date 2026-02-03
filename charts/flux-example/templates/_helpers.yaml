{{/*
Expand the name of the chart.
*/}}
{{- define "flux-bootstrap.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "flux-bootstrap.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "flux-bootstrap.labels" -}}
helm.sh/chart: {{ include "flux-bootstrap.name" . }}
{{ include "flux-bootstrap.selectorLabels" . }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "flux-bootstrap.selectorLabels" -}}
app.kubernetes.io/name: {{ include "flux-bootstrap.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
