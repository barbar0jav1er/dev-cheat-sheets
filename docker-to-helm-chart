# Guía Completa: De Docker a Helm Chart Profesional

*De imagen Docker básica a chart production-ready paso a paso*

---

## 📋 Tabla de Contenidos

1. [Conceptos Fundamentales](#conceptos-fundamentales)
2. [Análisis de la Aplicación](#análisis-de-la-aplicación)
3. [Estructura del Chart](#estructura-del-chart)
4. [Desarrollo Paso a Paso](#desarrollo-paso-a-paso)
5. [Patrones y Mejores Prácticas](#patrones-y-mejores-prácticas)
6. [Testing y Validación](#testing-y-validación)
7. [Deployment y Troubleshooting](#deployment-y-troubleshooting)

---

## 🎯 Conceptos Fundamentales

### ¿Qué es Helm?

**Helm** es el "package manager" de Kubernetes que permite empaquetar, versionar y distribuir aplicaciones como **charts**.

```
Docker Image → Kubernetes Manifests → Helm Chart → Production Ready
```

### Arquitectura de un Helm Chart

```
chart/
├── Chart.yaml          # Metadatos del chart
├── values.yaml         # Configuración por defecto
└── templates/          # Plantillas de Kubernetes
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── _helpers.tpl    # Funciones reutilizables
    └── NOTES.txt       # Instrucciones post-instalación
```

### Ventajas de Helm vs YAML Estático

| Aspecto | YAML Estático | Helm Chart |
|---------|---------------|------------|
| **Configuración** | Hardcodeada | Parametrizable |
| **Reutilización** | Copy-paste | Instalar con diferentes valores |
| **Mantenimiento** | Manual en múltiples archivos | Centralizado en helpers |
| **Versionado** | Git commits | Semantic versioning |
| **Rollbacks** | Manual | `helm rollback` |

---

## 🔍 Análisis de la Aplicación

### Paso 1: Entender los Requisitos

Antes de crear el chart, analiza qué necesita tu aplicación:

#### **Pi-hole como ejemplo:**
- **Servicios**: DNS (puerto 53) + Web Interface (puerto 80)
- **Persistencia**: Configuración (`/etc/pihole`) + DNS config (`/etc/dnsmasq.d`)
- **Seguridad**: Password como Secret, no plain text
- **Red**: LoadBalancer para DNS, Ingress para web
- **Variables**: Timezone, DNS upstreams, password

#### **Template de análisis para cualquier aplicación:**

```yaml
# Checklist de análisis
aplicacion: "nombre-app"
requisitos:
  puertos:
    - "80/tcp (web)"
    - "443/tcp (https)"
  persistencia:
    - "/app/data (configuración)"
    - "/app/logs (opcional)"
  secrets:
    - "database-password"
    - "api-keys"
  variables_env:
    - "DATABASE_URL"
    - "DEBUG_MODE"
  servicios:
    - "ClusterIP para web"
    - "LoadBalancer si necesita acceso externo"
  recursos:
    cpu: "100m-500m"
    memoria: "128Mi-512Mi"
```

---

## 📁 Estructura del Chart

### Jerarquía de archivos completa:

```
pihole/
├── Chart.yaml                 # ← Metadatos: nombre, versión, descripción
├── values.yaml                # ← Configuración centralizada
├── templates/                 # ← Plantillas de Kubernetes
│   ├── _helpers.tpl           # ← Funciones reutilizables
│   ├── deployment.yaml        # ← Contenedor principal
│   ├── service-dns.yaml       # ← LoadBalancer para DNS
│   ├── service-web.yaml       # ← ClusterIP para web
│   ├── ingress.yaml           # ← Exposición HTTPS
│   ├── pvc.yaml               # ← Almacenamiento persistente
│   ├── secret.yaml            # ← Passwords seguros
│   ├── serviceaccount.yaml    # ← Identidad de la aplicación
│   └── NOTES.txt              # ← Instrucciones al usuario
└── charts/                    # ← Dependencies (vacío para charts simples)
```

---

## 🚀 Desarrollo Paso a Paso

### Metodología: "Values-First, Templates-Second"

**Patrón seguido en cada componente:**
1. 📝 **Definir configuración** en `values.yaml`
2. 🏗️ **Crear template** que lee esa configuración
3. 🧪 **Probar** con diferentes valores
4. ♻️ **Refactorizar** con helpers si es necesario

---

### Fase 1: Chart Base

#### Step 1.1: Chart.yaml
```yaml
apiVersion: v2
name: pihole
description: Pi-hole DNS ad-blocker chart
type: application
version: 0.1.0                    # Versión del CHART
appVersion: "2024.07.0"          # Versión de la APLICACIÓN

maintainers:
  - name: Tu Nombre
    email: tu-email@domain.com
keywords:
  - dns
  - ad-blocker
  - homelab
```

#### Step 1.2: values.yaml inicial
```yaml
image:
  repository: pihole/pihole
  tag: "2024.07.0"
  pullPolicy: IfNotPresent

# Overrides de nombres
nameOverride: ""
fullnameOverride: ""

# Labels personalizables
commonLabels: {}
podLabels: {}
```

---

### Fase 2: Deployment Básico

#### Step 2.1: Deployment con configuración básica
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "pihole.fullname" . }}
  labels:
    {{- include "pihole.labels" . | nindent 4 }}
spec:
  replicas: 1
  selector:
    matchLabels:
      {{- include "pihole.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "pihole.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 53
          name: dns-udp
          protocol: UDP
        - containerPort: 80
          name: web
```

#### Step 2.2: Helpers fundamentales
```yaml
# templates/_helpers.tpl

{{/*
Nombre completo del recurso
*/}}
{{- define "pihole.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{/*
Labels comunes para todos los recursos
*/}}
{{- define "pihole.labels" -}}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
app.kubernetes.io/name: {{ include "pihole.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- with .Values.commonLabels }}
{{ toYaml . }}
{{- end }}
{{- end }}

{{/*
Labels de selector (consistentes entre deployment y services)
*/}}
{{- define "pihole.selectorLabels" -}}
app.kubernetes.io/name: {{ include "pihole.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

---

### Fase 3: Servicios de Red

#### Step 3.1: Configuración en values.yaml
```yaml
services:
  dns:
    enabled: true
    type: LoadBalancer
    loadBalancerIP: ""           # IP específica para MetalLB
  
  web:
    enabled: true
    type: ClusterIP
    port: 80
```

#### Step 3.2: Templates de Services
```yaml
# templates/service-dns.yaml
{{- if .Values.services.dns.enabled }}
apiVersion: v1
kind: Service
metadata:
  name: {{ include "pihole.fullname" . }}-dns
  labels:
    {{- include "pihole.labels" . | nindent 4 }}
spec:
  type: {{ .Values.services.dns.type }}
  {{- if and (eq .Values.services.dns.type "LoadBalancer") .Values.services.dns.loadBalancerIP }}
  loadBalancerIP: {{ .Values.services.dns.loadBalancerIP }}
  {{- end }}
  ports:
  - name: dns-tcp
    port: 53
    targetPort: 53
    protocol: TCP
  - name: dns-udp
    port: 53
    targetPort: 53
    protocol: UDP
  selector:
    {{- include "pihole.selectorLabels" . | nindent 4 }}
{{- end }}
```

---

### Fase 4: Exposición HTTPS

#### Step 4.1: Configuración Ingress
```yaml
# values.yaml
ingress:
  enabled: true
  className: "traefik"
  host: "pihole.local"
  annotations: {}
  tls:
    enabled: true
    secretName: "pihole-tls"
```

#### Step 4.2: Template Ingress
```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "pihole.fullname" . }}-web
  labels:
    {{- include "pihole.labels" . | nindent 4 }}
  annotations:
    {{- if .Values.ingress.tls.enabled }}
    traefik.ingress.kubernetes.io/router.tls: "true"
    {{- end }}
    {{- range $key, $value := .Values.ingress.annotations }}
    {{ $key | quote }}: {{ $value | quote }}
    {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  rules:
  - host: {{ .Values.ingress.host | quote }}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {{ include "pihole.fullname" . }}-web
            port:
              number: {{ .Values.services.web.port }}
  {{- if .Values.ingress.tls.enabled }}
  tls:
  - hosts:
    - {{ .Values.ingress.host | quote }}
    secretName: {{ .Values.ingress.tls.secretName }}
  {{- end }}
{{- end }}
```

---

### Fase 5: Persistencia de Datos

#### Step 5.1: Configuración de volúmenes
```yaml
# values.yaml
persistence:
  # Configuración principal
  config:
    enabled: true
    storageClass: ""
    size: "1Gi"
    accessModes:
      - ReadWriteOnce
    annotations: {}
  
  # Configuración DNS adicional
  dnsmasq:
    enabled: true
    storageClass: ""
    size: "500Mi"
    accessModes:
      - ReadWriteOnce
    annotations: {}
```

#### Step 5.2: PersistentVolumeClaims
```yaml
# templates/pvc.yaml
{{- if .Values.persistence.config.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "pihole.fullname" . }}-config
  labels:
    {{- include "pihole.labels" . | nindent 4 }}
spec:
  accessModes:
    {{- range .Values.persistence.config.accessModes }}
    - {{ . }}
    {{- end }}
  {{- if .Values.persistence.config.storageClass }}
  storageClassName: {{ .Values.persistence.config.storageClass | quote }}
  {{- end }}
  resources:
    requests:
      storage: {{ .Values.persistence.config.size }}
{{- end }}
```

#### Step 5.3: Conectar volúmenes al Deployment
```yaml
# En deployment.yaml, añadir:
        {{- if or .Values.persistence.config.enabled .Values.persistence.dnsmasq.enabled }}
        volumeMounts:
        {{- if .Values.persistence.config.enabled }}
        - name: config
          mountPath: /etc/pihole
        {{- end }}
        {{- if .Values.persistence.dnsmasq.enabled }}
        - name: dnsmasq
          mountPath: /etc/dnsmasq.d
        {{- end }}
        {{- end }}
      
      {{- if or .Values.persistence.config.enabled .Values.persistence.dnsmasq.enabled }}
      volumes:
      {{- if .Values.persistence.config.enabled }}
      - name: config
        persistentVolumeClaim:
          claimName: {{ include "pihole.fullname" . }}-config
      {{- end }}
      {{- end }}
```

---

### Fase 6: Gestión Segura de Secrets

#### Step 6.1: Configuración de passwords
```yaml
# values.yaml
password:
  # Crear Secret automáticamente
  create: true
  value: "changeme123"
  
  # O usar Secret existente
  existingSecret: ""
  existingSecretKey: "admin-password"
```

#### Step 6.2: Template de Secret
```yaml
# templates/secret.yaml
{{- if .Values.password.create }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "pihole.fullname" . }}-secret
  labels:
    {{- include "pihole.labels" . | nindent 4 }}
type: Opaque
data:
  admin-password: {{ .Values.password.value | b64enc | quote }}
{{- end }}
```

#### Step 6.3: Helpers para Secrets
```yaml
# En _helpers.tpl
{{/*
Nombre del Secret que contiene el password
*/}}
{{- define "pihole.secretName" -}}
{{- if .Values.password.create -}}
{{- printf "%s-secret" (include "pihole.fullname" .) -}}
{{- else -}}
{{- .Values.password.existingSecret -}}
{{- end -}}
{{- end }}

{{/*
Key del password en el Secret
*/}}
{{- define "pihole.secretKey" -}}
{{- if .Values.password.create -}}
admin-password
{{- else -}}
{{- .Values.password.existingSecretKey -}}
{{- end -}}
{{- end }}
```

#### Step 6.4: Usar Secret en Deployment
```yaml
# En deployment.yaml, env section:
        env:
        - name: FTLCONF_webserver_api_password
          valueFrom:
            secretKeyRef:
              name: {{ include "pihole.secretName" . }}
              key: {{ include "pihole.secretKey" . }}
```

---

### Fase 7: Resources y ServiceAccount

#### Step 7.1: Resource limits
```yaml
# values.yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

#### Step 7.2: ServiceAccount
```yaml
# values.yaml
serviceAccount:
  create: true
  name: ""
  annotations: {}
```

```yaml
# templates/serviceaccount.yaml
{{- if .Values.serviceAccount.create -}}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "pihole.serviceAccountName" . }}
  labels:
    {{- include "pihole.labels" . | nindent 4 }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
{{- end }}
```

---

### Fase 8: Experiencia de Usuario

#### Step 8.1: NOTES.txt dinámico
```yaml
# templates/NOTES.txt
🎉 Pi-hole has been deployed successfully!

📊 Release: {{ .Release.Name }} ({{ .Release.Namespace }})

{{- if .Values.ingress.enabled }}
🌐 WEB INTERFACE:
   {{- if .Values.ingress.tls.enabled }}
   🔒 URL: https://{{ .Values.ingress.host }}/admin
   {{- else }}
   🔓 URL: http://{{ .Values.ingress.host }}/admin
   {{- end }}
   🔑 Password: {{ .Values.password.value | quote }}
{{- else }}
🌐 WEB INTERFACE (Ingress disabled):
   Run: kubectl port-forward service/{{ include "pihole.fullname" . }}-web 8080:80
   Then visit: http://localhost:8080/admin
{{- end }}

{{- if .Values.services.dns.enabled }}
📡 DNS SERVER:
   Get external IP: kubectl get service {{ include "pihole.fullname" . }}-dns --watch
{{- end }}

📋 NEXT STEPS:
1. Access the web interface and configure blocklists
2. Set Pi-hole as your DNS server
3. Test DNS blocking: nslookup doubleclick.net <dns-server-ip>
```

---

## 🎨 Patrones y Mejores Prácticas

### Pattern 1: Values-First Development

**Siempre empezar por values.yaml:**
```yaml
# ❌ MAL: Template primero
# templates/deployment.yaml con valores hardcodeados

# ✅ BIEN: Values primero
# values.yaml con configuración
# templates/deployment.yaml que lee values
```

### Pattern 2: Helpers para Consistencia

**Usar helpers para nombres y labels:**
```yaml
# ❌ MAL: Nombres hardcodeados
name: pihole-config
app: pihole

# ✅ BIEN: Helpers consistentes
name: {{ include "pihole.fullname" . }}-config
{{- include "pihole.selectorLabels" . | nindent 4 }}
```

### Pattern 3: Configuración Opcional

**Todo debe ser opcional y configurable:**
```yaml
# Patrón para recursos opcionales
{{- if .Values.feature.enabled }}
# recurso solo si está habilitado
{{- end }}

# Patrón para configuración condicional
{{- if and (eq .Values.service.type "LoadBalancer") .Values.service.loadBalancerIP }}
loadBalancerIP: {{ .Values.service.loadBalancerIP }}
{{- end }}
```

### Pattern 4: Control de Espacios

```yaml
# En helpers: siempre {{- -}}
{{- define "app.name" -}}
contenido
{{- end -}}

# En templates: solo {{-
{{- if condition }}
contenido
{{- end }}

# Al usar helpers: sin espacios extra
name: {{ include "app.name" . }}
```

### Pattern 5: Seguridad por Defecto

```yaml
# ❌ MAL: Passwords en plain text
adminPassword: "changeme123"

# ✅ BIEN: Passwords como Secrets
password:
  create: true
  value: "changeme123"  # Solo durante instalación
```

---

## 🧪 Testing y Validación

### Comandos Esenciales

#### Validación de sintaxis:
```bash
# Lint del chart completo
helm lint .

# Validar templates
helm template app . --validate

# Ver output generado
helm template app . --debug
```

#### Testing con diferentes configuraciones:
```bash
# Configuración mínima
helm template app . \
  --set ingress.enabled=false \
  --set persistence.enabled=false

# Configuración completa
helm template app . \
  --set ingress.host="app.production.com" \
  --set resources.limits.memory="2Gi"

# Con valores personalizados
helm template app . -f values-production.yaml
```

#### Dry-run antes de deploy:
```bash
helm install app . --dry-run --debug
```

### Checklist de Validación

- [ ] ✅ `helm lint .` sin errores
- [ ] ✅ Templates generan YAML válido
- [ ] ✅ Labels consistentes entre recursos
- [ ] ✅ Selectors coinciden entre Services y Deployments
- [ ] ✅ Nombres únicos con helpers
- [ ] ✅ Recursos opcionales funcionan
- [ ] ✅ Secrets manejados correctamente
- [ ] ✅ NOTES.txt útil y dinámico

---

## 🚀 Deployment y Troubleshooting

### Proceso de Deploy

#### 1. Validación pre-deploy:
```bash
helm template myapp . > /tmp/myapp-test.yaml
kubectl apply --dry-run=client -f /tmp/myapp-test.yaml
```

#### 2. Deploy con monitoreo:
```bash
helm install myapp . --wait --timeout=5m
kubectl get pods -l app.kubernetes.io/instance=myapp --watch
```

#### 3. Verificación post-deploy:
```bash
helm status myapp
kubectl get all -l app.kubernetes.io/instance=myapp
```

### Troubleshooting Common Issues

#### Error: "YAML parse error"
```bash
# Debug: ver línea exacta del error
helm template app . --debug

# Causa común: indentación incorrecta o falta de espacios después de ":"
```

#### Error: "selector does not match template labels"
```bash
# Verificar que selectorLabels coincidan
helm template app . | grep -A5 "matchLabels"
helm template app . | grep -A10 "template:" | grep -A5 "labels"
```

#### Pod no arranca:
```bash
kubectl describe pod -l app.kubernetes.io/name=myapp
kubectl logs -l app.kubernetes.io/name=myapp

# Causas comunes:
# - Secret no existe o key incorrecta
# - PVC no bound
# - Image pull error
# - Resources insuficientes
```

#### Service sin endpoints:
```bash
kubectl get endpoints
kubectl get pods -o wide

# Causa: selector labels no coinciden
```

---

## 📚 Recursos y Referencias

### Documentación Oficial
- [Helm Charts Guide](https://helm.sh/docs/chart_template_guide/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Go Templates](https://pkg.go.dev/text/template)
- [Sprig Functions](https://masterminds.github.io/sprig/)

### Herramientas Útiles
```bash
# Helm plugins recomendados
helm plugin install https://github.com/chartmuseum/helm-push
helm plugin install https://github.com/helm/helm-2to3

# Validación adicional
kubeval output.yaml          # Validar YAML de Kubernetes
helm-schema-gen values.yaml  # Generar JSON schema para values
```

### Templates de Chart Base
```bash
# Crear chart nuevo con estructura completa
helm create newapp

# Usar chart de Pi-hole como template
cp -r pihole newapp
# Modificar Chart.yaml, values.yaml y templates
```

---

## 🎯 Resumen de la Metodología

### Fases de Desarrollo
1. **Análisis** → Entender requisitos de la aplicación
2. **Estructura** → Crear chart base con Chart.yaml y values.yaml
3. **Core** → Deployment básico con helpers fundamentales
4. **Servicios** → Exposición de red (Services + Ingress)
5. **Persistencia** → Almacenamiento de datos
6. **Seguridad** → Secrets y ServiceAccounts
7. **Producción** → Resources, labels, NOTES.txt
8. **Testing** → Validación y troubleshooting

### Principios Clave
- **Values-First**: Configuración antes que templates
- **Helpers para Consistencia**: Un solo lugar para lógica común
- **Todo Opcional**: Cada feature debe poder deshabilitarse
- **Security by Default**: Secrets, resources limits, labels estándar
- **User Experience**: NOTES.txt útil y documentación clara

### Resultado Final
Un Helm Chart **production-ready** que es:
- 🔧 **Configurable** (múltiples entornos)
- 🔒 **Seguro** (secrets, resources)
- 📦 **Reutilizable** (múltiples instalaciones)
- 🚀 **Profesional** (labels estándar, helpers)
- 👥 **User-friendly** (NOTES.txt, validación)

---

*Esta guía te permite convertir cualquier imagen Docker en un Helm Chart profesional siguiendo patrones establecidos y mejores prácticas de la industria.*
