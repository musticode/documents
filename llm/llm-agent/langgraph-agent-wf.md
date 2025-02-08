# LangGraph - workflow, agent


- Workflow, Büyük Dil Modelleri’nin ve Agent’lerin önceden tanımlanmış akışlar, yollar üzerinden düzenlediği sistemlerdir. Agent ise bu modellerin kendi süreçlerini ve araçları (tool) dinamik olarak yürüttüğü, görevlerini nasıl gerçekleştireceği konusunda kontrolü elinde tuttuğu sistemlerdir.
- workflow patterns:
    - Prompt Chaining
    - Parallelization
    - Routing
    - Orchestrator-workers
    - Evaluator-optimizer

### Prompt chaining
bir görevi adım adım ilerleterek her LLM isteğinin önceki çıktıyı işlemesini sağlar. Sürecin doğru ilerlediğinden emin olabilmek için süreçte ara adımlar, aşağıdaki görselde “GATE” olarak örneğini verdiğim yapılar, tanımlanabilir. Süreç, sabit alt görevlere net bir biçimde bölünebiliyorsa kullanılabilir.
- [prompt chaining example code](https://github.com/buraketmen/langchain-agent/tree/main/prompt-chaining)

### Workflow: Parallelization
LLM’ler bir görevi parçalara ayırıp aynı anda işleyip çıktılarını programatik olarak birleştirebilir. Bir görevi bağımsız alt görevlere bölerek de işleyebiliriz, farklı çıktılar elde edebilmek için aynı görevi birden fazla kez çalıştırarak da. Bu akışı, hız için alt görevleri paralel olarak çalıştırabileceğimiz ya da farklı bakış açılarına ihtiyaç duyabileceğimiz durumlarda kullanabiliriz.
- [paralellization](https://github.com/buraketmen/langchain-agent/tree/main/parallelization)

### Routing
Routing, sorumlulukları ayırmaya olanak tanır. Girdiyi kategorilendirir ve kategorilendirilen girdiye özel süreçleri yürütür. Farklı kategorilere ayrılabilen ve her kategorinin ayrı ele alınmasının daha iyi sonuç vereceği karmaşık görevler için uygundur. Girdileri kategorize etmek için hem geleneksel bir sınıflandırma modeli hem de LLM kullanılabilir.
[routing](https://github.com/buraketmen/langchain-agent/tree/main/routing)


### Workflow Orchestrator-Workers
Merkezi bir LLM’in, görevleri dinamik olarak bölerek işçi LLM’lere devrettiği ve sonuçlarını işlediği bir yapıdır. Yapılması istenen bir görev için gereken alt görevlerin önceden tahmin edilemediği, karmaşık akışlar için idealdir. Parallelization’a benzese de temel farkı esnekliktir. Parallelization’da alt görevler önceden belirlenirken bu akışta girdiye bağlı olarak oluşturulur. Dinamik olarak oluşturulduğu için de düşük parametreli modellerde stabil olmayabilir.
[orchestrator-worker](https://github.com/buraketmen/langchain-agent/tree/main/orchestrator-worker)

### Workflow: Evaluator-Optimizer

Evaluator-optimizer, bir girdiden sonuç üretilirken sonucun değerlendirip geri bildirimlerde bulunulan bir döngüye dayanır. Yinelemeli iyileştirmeler ile ölçülebilir değerler elde edilebileceğinde etkilidir.

[evaluator optimizer](https://github.com/buraketmen/langchain-agent/tree/main/evaluator-optimizer)

## Agent

Agent’ler, isteğe bağlı görevleri yerine getirebilir ancak uygulamaları nispeten basittir. Çoğu zaman çevresel geri bildirimlere dayalı olarak kullanılan LLM’lerden oluşurlar. Gerekli adım sayısının tahmin edilmesinin zor olduğu, sabit bir yolun yürütülemeyeceği açık uçlu problemler için kullanılabilir.
[agent](https://github.com/buraketmen/langchain-agent/tree/main/agent)


örnek kodlar : https://github.com/buraketmen/langchain-agent

kaynak : 

    https://www.anthropic.com/research/building-effective-agents
    https://langchain-ai.github.io/langgraph/tutorials/workflows/
    https://github.com/buraketmen/langchain-agent