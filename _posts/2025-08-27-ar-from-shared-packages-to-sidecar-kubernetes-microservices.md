---
layout: post
title: "من الـ Shared Packages إلى الـ Sidecar: كيف يساعد Kubernetes في تبسيط الـ Microservices"
excerpt: "الـ Microservices بقت من أشهر الطرق اللي الشركات بتبني بيها أنظمة كبيرة ومعقدة. بدل ما يبقى عندك تطبيق ضخم وصعب تعديله، بتقسمه لخدمات صغيرة مستقلة. كل خدمة ليها وظيفتها، وده بيخلي التطوير أسرع و أسهل في الصيانة. بس الحقيقة الموضوع مش بالبساطة دي. فيه مشاكل كتير بتظهر لما تيجي تدير الـ microservices، خصوصًا في الحاجات اللي مشتركة بينهم كلهم: زي الـ logging، الـ monitoring، الـ security، أو حتى التعامل مع الـ configs… ده اللي هانحاول نلمسه في المقالة دي وازاي k8s ممكن يساعد."
tags:
  - Kubernetes
  - k8s
  - Microservices
  - Sidecar
  - كبسولات_k8s
  - تصميم_الأنظمة
lang: ar
dir: rtl
---

## مقدمة

الـ Microservices بقت من أشهر الطرق اللي الشركات بتبني بيها أنظمة كبيرة ومعقدة. بدل ما يبقى عندك تطبيق ضخم وصعب تعديله، بتقسمه لخدمات صغيرة مستقلة. كل خدمة ليها وظيفتها، وده بيخلي التطوير أسرع و أسهل في الصيانة.

بس الحقيقة الموضوع مش بالبساطة دي. فيه مشاكل كتير بتظهر لما تيجي تدير الـ microservices، خصوصًا في الحاجات اللي مشتركة بينهم كلهم: زي الـ logging، الـ monitoring، الـ security، أو حتى التعامل مع الـ configs… ده اللي هانحاول نلمسه في المقالة دي وازاي k8s ممكن يساعد.

## دور Kubernetes مع الـ Microservices

هنا بيظهر دور Kubernetes (k8s). هو مش مجرد أداة بتشغّل containers، هو عامل زي نظام تشغيل للسيرفر. بيسهّل:
- تشغيل مئات أو آلاف الـ containers.
- الـ scaling التلقائي للخدمات.
- إدارة التوزيع (deployment) والـ rollback بسهولة.

بمعنى آخر، Kubernetes بيوفرلك الأرضية اللي تقدر تبني عليها الـ microservices وتديرها من غير ما تدخل في تفاصيل كتيرة.

## فكرة الـ Sidecar

واحدة من أقوى الأفكار اللي طلعت مع Kubernetes هي فكرة الـ sidecar. 
الـ sidecar هو container صغير بيشتغل جنب الـ microservice جوا نفس الـ pod.

الميزة هنا إنك تقدر تحط فيه أي cross-cutting concern (CCC) زي:
- الـ Logging
- الـ Authentication
- الـ Authorization
- الـ Caching
- الـ Monitoring
- الـ Messaging 

بدل ما تحشي الـ microservice بكل الحاجات دي وتكبرها وتعقدها من غير لازمة، ممكن تعزلها في sidecar منفصل، والـ service تفضل مركزة على شغلها الأساسي بس.

## بناء Cross-Cutting-Concern Image

بدل ما تعمل shared packages (زي NuGet أو npm) وتوزعها على كل فريق، ممكن تبني صورة (image) فيها الـ cross-cutting functionality. 

الميزة في الطريقة دي إنها:
- إعادة الاستخدام (reusable) على مستوى الشركة كلها.
- مش مرتبطة بلغة برمجة معينة → تنفع مع .NET، Node.js، Java أو أي تكنولوجيا.
- معزولة في كبسولة (encapsulated) → كل التفاصيل متقفلة جوه image واحدة ليها APIs واضحة.
- أي إعدادات (زي telemetry أو tracing) بتتبني جوا الـ image نفسها، وده بيخلي الـ microservice أنضف ومركزة أكتر.

## مقارنة مع الـ Shared Packages

لو استخدمت shared packages:
- أي تعديل صغير → لازم rebuild لكل التطبيقات اللي بتستخدمها.
- مشاكل dependencies والـ versioning المشهورة واللي بتتكرر كتير.

لكن مع الـ sidecar approach:
- بتعمل update للـ ccc image مرة واحدة.
- التحديث بيتعمله حقن injection بالـ CI/CD (زي ArgoCD) من غير ما تعيد بناء كل microservice.
- بتبقى عامل زي مكوّن خارجي بيتحدث بشكل مستقل.

## فوائد الـ Sidecar Approach

- الخدمات microservices أخف → متركزة على شغلها الأساسي بس.
- إعادة الاستخدام → نفس الـ image تشتغل مع أي خدمة.
- سهولة التحديث → تحديث واحد يتطبق على الكل.
- مرونة التكنولوجيا → مش فارقة اللغة أو الـ framework.
- تقليل التكرار → مش محتاج تكتب نفس الكود في كل microservice.
- الإعدادات معزولة → configs زي telemetry والـ monitoring بتبقى جوه الـ sidecar بدل ما تلوث الـ microservice.

## الخاتمة

الـ sidecar مش حل لكل حاجة، بس في معظم الحالات هو أفضل بكتير من فكرة الـ shared packages.
لما تجمع Kubernetes مع الـ sidecar pattern، هتلاقي نفسك بتبني بنية تحتية حديثة ومرنة بتخلّي شغل الـ microservices أبسط وأسرع وأنضف.
