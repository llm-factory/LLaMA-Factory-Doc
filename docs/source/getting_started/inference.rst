推理
==========================

LLaMA-Factory 支持多种推理方式。

您可以使用 ``llamafactory-cli chat inference_config.yaml`` 或 ``llamafactory-cli webchat inference_config.yaml`` 进行推理与模型对话。对话时配置文件只需指定原始模型 ``model_name_or_path`` 和 ``template`` ，并根据是否是微调模型指定 ``adapter_name_or_path`` 和 ``finetuning_type``。

如果您希望向模型输入大量数据集并保存推理结果，您可以启动 :ref:`vllm <vllm>` 推理引擎对大量数据集进行快速的批量推理。您也可以通过 :ref:`部署 api <api>` 服务的形式通过 api 调用来进行批量推理。

默认情况下，模型推理将使用 Huggingface 引擎。 您也可以指定 ``infer_backend: vllm`` 以使用 vllm 推理引擎以获得更快的推理速度。 


.. note::
    使用任何方式推理时，模型 ``model_name_or_path`` 需要存在且与 ``template`` 相对应。

原始模型推理配置
----------------------------
对于原始模型推理， ``inference_config.yaml`` 中 只需指定原始模型 ``model_name_or_path`` 和 ``template`` 即可。

.. code-block:: yaml

    ### examples/inference/llama3.yaml
    model_name_or_path: meta-llama/Meta-Llama-3-8B-Instruct
    template: llama3
    infer_backend: huggingface #choices： [huggingface, vllm]  


微调模型推理配置
------------------------------
对于微调模型推理，除原始模型和模板外，还需要指定适配器路径 ``adapter_name_or_path`` 和微调类型 ``finetuning_type``。

.. code-block:: yaml

    ### examples/inference/llama3_lora_sft.yaml
    model_name_or_path: meta-llama/Meta-Llama-3-8B-Instruct
    adapter_name_or_path: saves/llama3-8b/lora/sft
    template: llama3
    finetuning_type: lora
    infer_backend: huggingface #choices： [huggingface, vllm]


多模态模型
----------------------

对于多模态模型，您可以运行以下指令进行推理。

.. code-block:: bash

    llamafactory-cli webchat examples/inference/llava1_5.yaml

``examples/inference/llava1_5.yaml`` 的配置示例如下：

.. code-block:: yaml

    model_name_or_path: llava-hf/llava-1.5-7b-hf
    template: vicuna
    infer_backend: huggingface #choices： [huggingface, vllm]
    


.. _批量推理:

批量推理
-------------------------


.. _vllm:

数据集
~~~~~~~~~~~~~~~~~~~~~~~
您可以通过以下指令启动 vllm 推理引擎并使用数据集进行批量推理：

.. code-block:: python

    python scripts/vllm_infer.py --model_name_or_path path_to_merged_model --dataset alpaca_en_demo


.. _api:

api
~~~~~~~~~~~~~~~~~
如果您需要使用 api 进行批量推理，您只需指定模型、适配器（可选）、模板、微调方式等信息。

下面是一个配置文件的示例：

.. code-block:: yaml

    ### examples/inference/llama3_lora_sft.yaml
    model_name_or_path: meta-llama/Meta-Llama-3-8B-Instruct
    adapter_name_or_path: saves/llama3-8b/lora/sft
    template: llama3
    finetuning_type: lora


下面是一个启动并调用 api 服务的示例：

您可以使用 ``API_PORT=8000 CUDA_VISIBLE_DEVICES=0 llamafactory-cli api examples/inference/llama3_lora_sft.yaml`` 启动 api 服务并运行以下示例程序进行调用：

.. code-block:: python

    # api_call_example.py
    from openai import OpenAI
    client = OpenAI(api_key="0",base_url="http://0.0.0.0:8000/v1")
    messages = [{"role": "user", "content": "Who are you?"}]
    result = client.chat.completions.create(messages=messages, model="meta-llama/Meta-Llama-3-8B-Instruct")
    print(result.choices[0].message)




