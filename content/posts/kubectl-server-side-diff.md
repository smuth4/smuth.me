+++
title = "'Ghost' changes in kubernetes' server-side apply"
date = "2022-02-20"
tags = ["kubernetes"]
+++

Starting in version 1.22, [server-side apply](https://kubernetes.io/docs/reference/using-api/server-side-apply/) has become officially "stable" in kubernetes, and in most is truly an upgrade from client-side applies. It respects fields that other server-side tools mark as controlled, and allows for applying larger and more complex objects.

However, there are a couple "gotchas" to be aware of. Most are either big loud errors, or called out explicitly in the documentation, but one thing I can across while working with server-side apply diffs (N.B.: client-side diffs are also sent to the server in new versions of kubernetes, it's a little confusing). Every time I went to apply a change, certain objects, particularly deployments, would have a timestamp in `metadata.managedFields` which would always change even if nothing else did:

```diff
diff -u -N /tmp/LIVE-902559977/rbac.authorization.k8s.io.v1.ClusterRole..awx-operator /tmp/MERGED-2370271049/rbac.authorization.k8s.io.v1.ClusterRole..awx-operator
--- /tmp/LIVE-902559977/rbac.authorization.k8s.io.v1.ClusterRole..awx-operator  2022-02-20 17:25:02.204297070 -0500
+++ /tmp/MERGED-2370271049/rbac.authorization.k8s.io.v1.ClusterRole..awx-operator       2022-02-20 17:25:02.204297070 -0500
@@ -27,7 +27,7 @@
       f:rules: {}
     manager: tanka
     operation: Apply
-    time: "2022-02-20T21:55:47Z"
+    time: "2022-02-20T22:25:02Z"
   - apiVersion: rbac.authorization.k8s.io/v1
     fieldsType: FieldsV1
     fieldsV1:
```

The tool I'm using is [tanka](https://tanka.dev/), but the same thing shows up when manually diffing. Even switching back to client-side diffing didn't reveal anything unexpected. After spending way too long staring at the generated YML against the live YAML, I found that most of these instances were due to kubernetes converting values, with some example as follows:

* `'1024Mi'` -> `'1Gi'`, in requests/limits.memory
* `'2000m'` -> `'2'`, in requests/limits.cpu

There were also some fields that also were removed from the live objects when set to null (e.g. `spec.selector` in services) which also caused the same timestamp change. In all these cases, the server-side apply was smart enough to determine that nothing actually changed, but since the fields were technically different between the new and live object, the timestamp was still updated.
