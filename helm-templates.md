# Helm Templates Cheatsheet

## 🎯 Sintaxis Básica

### Variables y Contexto
```yaml
{{ .Values.variable }}           # Variable de values.yaml
{{ .Chart.Name }}               # Nombre del chart
{{ .Chart.Version }}            # Versión del chart
{{ .Chart.AppVersion }}         # Versión de la aplicación
{{ .Release.Name }}             # Nombre del release
{{ .Release.Namespace }}        # Namespace del release
{{ .Release.Service }}          # Servicio de instalación (Helm)
{{ .Capabilities.KubeVersion }} # Versión de Kubernetes
```

### Acceso a valores anidados
```yaml
# values.yaml:
# image:
#   repository: nginx
#   tag: "1.21"
#   pullPolicy: Always

{{ .Values.image.repository }}  # nginx
{{ .Values.image.tag }}         # 1.21
{{ .Values.image.pullPolicy }}  # Always
```

## 🔧 Funciones Útiles

### Strings
```yaml
{{ .Values.name | quote }}           # "value"
{{ .Values.name | squote }}          # 'value'
{{ .Values.name | upper }}           # VALOR
{{ .Values.name | lower }}           # valor
{{ .Values.name | title }}           # Valor
{{ .Values.name | untitle }}         # valor
{{ .Values.name | repeat 3 }}        # valorvalorvalor
```

### Manipulación de texto
```yaml
{{ .Values.name | trunc 63 }}        # Corta a 63 caracteres
{{ .Values.name | trimSuffix "-" }}  # Quita "-" del final
{{ .Values.name | trimPrefix "pre" }} # Quita "pre" del inicio
{{ .Values.name | trim }}            # Quita espacios
{{ .Values.name | replace " " "-" }} # Reemplaza espacios por -
```

### Defaults y validación
```yaml
{{ .Values.tag | default "latest" }}           # Si vacío, usa "latest"
{{ .Values.tag | default .Chart.AppVersion }}  # Si vacío, usa AppVersion
{{ .Values.name | required "name is required" }} # Obliga a tener valor
{{ .Values.count | toString }}                 # Convierte a string
{{ .Values.enabled | ternary "yes" "no" }}     # Si/No basado en boolean
```

## 🔀 Pipes y Encadenamiento

```yaml
{{ .Values.image.tag | default .Chart.AppVersion | quote }}
# 1. Si tag vacío → usa AppVersion
# 2. Resultado → envuelve en comillas

{{ .Values.name | lower | replace " " "-" | trunc 63 | trimSuffix "-" }}
# 1. Convierte a minúsculas
# 2. Reemplaza espacios por guiones
# 3. Corta a 63 caracteres
# 4. Quita guión final si existe
```

## ❓ Condicionales

### If básico
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
# ... resto del recurso
{{- end }}
```

### If con else
```yaml
{{- if .Values.image.tag }}
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
{{- else }}
image: {{ .Values.image.repository }}:{{ .Chart.AppVersion }}
{{- end }}
```

### Operadores de comparación
```yaml
{{- if eq .Values.env "production" }}     # Igual a
{{- if ne .Values.env "development" }}    # No igual a
{{- if gt .Values.replicas 1 }}           # Mayor que
{{- if lt .Values.replicas 10 }}          # Menor que
{{- if ge .Values.replicas 1 }}           # Mayor o igual
{{- if le .Values.replicas 10 }}          # Menor o igual
```

### Operadores lógicos
```yaml
{{- if and .Values.ingress.enabled .Values.ingress.tls }}        # AND
{{- if or .Values.debug .Values.development }}                   # OR
{{- if not .Values.production }}                                 # NOT
{{- if and (eq .Values.env "prod") (.Values.replicas | gt 1) }}  # Combinado
```

### Verificar si existe
```yaml
{{- if .Values.service }}              # Si service existe
{{- if hasKey .Values "database" }}    # Si key "database" existe
{{- if .Values.config.enabled }}       # Si config.enabled existe y es true
```

## 🔄 Loops (range)

### Range sobre lista
```yaml
# values.yaml:
# env:
#   - name: NODE_ENV
#     value: production
#   - name: PORT
#     value: "3000"

env:
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}
```

### Range sobre objeto/map
```yaml
# values.yaml:
# env:
#   NODE_ENV: production
#   PORT: "3000"
#   DEBUG: "false"

env:
{{- range $key, $value := .Values.env }}
- name: {{ $key }}
  value: {{ $value | quote }}
{{- end }}
```

### Range con índice
```yaml
{{- range $index, $item := .Values.items }}
- name: item-{{ $index }}
  value: {{ $item }}
{{- end }}
```

### Range con contexto padre
```yaml
{{- $fullName := include "myapp.fullname" . }}
{{- range .Values.configs }}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ $fullName }}-{{ .name }}  # Usa variable del contexto padre
{{- end }}
```

## 💾 Variables

### Asignar variables
```yaml
{{- $fullName := include "myapp.fullname" . }}
{{- $svcPort := .Values.service.port }}
{{- $releaseName := .Release.Name }}

name: {{ $fullName }}
port: {{ $svcPort }}
release: {{ $releaseName }}
```

### Variables en loops
```yaml
{{- range $index, $config := .Values.configs }}
{{- $configName := printf "%s-config-%v" (include "myapp.fullname" $) $index }}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ $configName }}
{{- end }}
```

## 📋 Templates y Includes

### Definir template en _helpers.tpl
```yaml
{{/*
Nombre completo de la aplicación
*/}}
{{- define "myapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{/*
Labels comunes
*/}}
{{- define "myapp.labels" -}}
helm.sh/chart: {{ include "myapp.chart" . }}
{{ include "myapp.selectorLabels" . }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Labels de selector
*/}}
{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### Usar templates
```yaml
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}

spec:
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
```

## 📝 Formateo y Indentación

### Indentación
```yaml
{{- include "myapp.labels" . | indent 4 }}    # Indenta 4 espacios
{{- include "myapp.labels" . | nindent 4 }}   # Nueva línea + indenta 4

# toYaml con indentación
resources:
{{- toYaml .Values.resources | nindent 2 }}

# Resultado:
resources:
  limits:
    cpu: 500m
    memory: 128Mi
```

### Conversión de datos
```yaml
{{- toYaml .Values.object | nindent 2 }}      # Objeto a YAML
{{- toJson .Values.object }}                  # Objeto a JSON
{{- .Values.list | join "," }}                # Lista a string separado por comas
{{- .Values.string | split "," }}             # String a lista
```

## 🎮 Control de Espacios

### Sin control (mantiene espacios y líneas)
```yaml
{{ if .Values.debug }}
debug: true
{{ end }}

# Resultado:
debug: true

```

### Con control de espacios
```yaml
{{- if .Values.debug }}
debug: true
{{- end }}

# Resultado:
debug: true
```

### Control específico
```yaml
{{- "texto" }}    # Quita espacio antes
{{ "texto" -}}    # Quita espacio después  
{{- "texto" -}}   # Quita espacios antes y después
```

## 🔍 Funciones Avanzadas

### Fechas
```yaml
{{ now | date "2006-01-02" }}           # Fecha actual: 2024-03-15
{{ now | date "2006-01-02T15:04:05Z" }} # Timestamp: 2024-03-15T10:30:45Z
```

### Listas
```yaml
{{ list "a" "b" "c" }}                  # Crear lista
{{ .Values.list | first }}             # Primer elemento
{{ .Values.list | last }}              # Último elemento
{{ .Values.list | initial }}           # Todos menos el último
{{ .Values.list | rest }}              # Todos menos el primero
{{ .Values.list | reverse }}           # Invertir lista
{{ .Values.list | uniq }}              # Elementos únicos
{{ .Values.list | sortAlpha }}         # Ordenar alfabéticamente
{{ has "item" .Values.list }}          # Verificar si contiene
```

### Diccionarios/Maps
```yaml
{{ dict "key1" "value1" "key2" "value2" }}     # Crear diccionario
{{ .Values.dict | keys }}                      # Obtener keys
{{ .Values.dict | values }}                    # Obtener values
{{ hasKey .Values.dict "mykey" }}              # Verificar si key existe
{{ pluck "key" .Values.listOfDicts }}          # Extraer valor de lista de dicts
```

## 📊 Ejemplos Prácticos

### Configuración de ambiente
```yaml
env:
{{- if eq .Values.environment "development" }}
- name: DEBUG
  value: "true"
- name: LOG_LEVEL
  value: "debug"
{{- else if eq .Values.environment "production" }}
- name: DEBUG
  value: "false" 
- name: LOG_LEVEL
  value: "info"
{{- end }}
{{- range $key, $value := .Values.env }}
- name: {{ $key }}
  value: {{ $value | quote }}
{{- end }}
```

### Recursos condicionales
```yaml
{{- if .Values.persistence.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  accessModes:
    {{- range .Values.persistence.accessModes }}
    - {{ . | quote }}
    {{- end }}
  resources:
    requests:
      storage: {{ .Values.persistence.size | quote }}
  {{- if .Values.persistence.storageClass }}
  storageClassName: {{ .Values.persistence.storageClass | quote }}
  {{- end }}
{{- end }}
```

### Múltiples ingress hosts
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  rules:
  {{- range .Values.ingress.hosts }}
  - host: {{ .host | quote }}
    http:
      paths:
      {{- range .paths }}
      - path: {{ .path }}
        pathType: {{ .pathType }}
        backend:
          service:
            name: {{ include "myapp.fullname" $ }}
            port:
              number: {{ $.Values.service.port }}
      {{- end }}
  {{- end }}
{{- end }}
```

## 🚀 Tips y Trucos

### Debug template
```bash
# Ver YAML generado
helm template myapp ./mychart --debug

# Con valores específicos
helm template myapp ./mychart --set image.tag=v2.0.0

# Con archivo de valores
helm template myapp ./mychart -f values-prod.yaml
```

### Validación
```bash
# Validar sintaxis
helm lint ./mychart

# Dry run
helm install myapp ./mychart --dry-run --debug
```

### Comentarios en templates
```yaml
{{/* 
Este es un comentario que no aparece 
en el YAML final 
*/}}

{{- /* Comentario con control de espacios */ -}}
```

### Template con múltiples documentos
```yaml
{{- range .Values.services }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .name }}
spec:
  # ... resto de la configuración
{{- end }}
```

## ⚠️ Errores Comunes

1. **Olvidar comillas**: `value: {{ .Values.string }}` → `value: {{ .Values.string | quote }}`
2. **Contexto en loops**: Usar `.` en vez de `$` para contexto root
3. **Espacios**: No controlar espacios con `{{-` y `-}}`
4. **Indentación**: Usar `indent` en vez de `nindent` cuando necesitas nueva línea
5. **Variables undefined**: Usar `default` para valores opcionales

---

**📚 Referencias útiles:**
- [Helm Template Guide](https://helm.sh/docs/chart_template_guide/)
- [Sprig Functions](https://masterminds.github.io/sprig/)
- [Go Templates](https://pkg.go.dev/text/template)
