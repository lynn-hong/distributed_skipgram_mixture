(2016-12-26 현재 작성 중입니다)
--


Distributed Multisense Word Embedding (Korean version documentation)
==========

Distributed Multisense Word Embedding(DMWE) tool은 Skip-Gram Mixture [1] 알고리즘을 분산 처리할 수 있도록 한 도구입니다. Microsoft의 Distributed Machine Learning Toolkit(DMTK) 구조를 따르고 있습니다.

Skip-Gram Mixture는 딥 러닝을 텍스트에 적용한 Word2Vec [2] 학습 방법 중 Skip-Gram을 확장한 것으로 다의어 문맥 학습이 되지 않는다는 Word2Vec의 치명적인 약점을 보완하였습니다.

DMTK에 대한 더 자세한 내용은 [http://www.dmtk.io](http://www.dmtk.io) 웹사이트를 참조하시기 바랍니다.



**본 문서는 https://github.com/Microsoft/distributed_skipgram_mixture.git 의 코드에 대한 한국어 버전 주석입니다.**

원본 Readme의 설명이 부족하고 실제 사용이 복잡하므로 이에 대한 추가 설명이 필요하다는 생각이 들었습니다. 본인이 학위논문 연구를 위해 본 코드를 사용하는 과정에서 발생했던 오류에 대한 해결책을 공유하고자
 합니다.  그리고 결과 파일을 해석하는 방법도 공유합니다.

**이하의 설명은 Linux Ubuntu 14.04를 기준으로 작성되었습니다. 원본 Readme에는 Windows용 설치에 대한 설명도 나와 있으며, [1] 의 실험도 Windows 환경에서 수행된 것으로 
보입니다. 본 문서는 Windows 환경에 대해선 다루지 않지만 많은 참고가 되실 거라고 생각합니다.**

Skip-Gram Mixture 알고리즘을 한국어에 적용한 연구는 [3] 학위논문을 참조 바랍니다.


코드 현 상태 및 구조 (Status and Structure)
----------
DMWE tool은 실질적으로 "deprecated(더 이상 버전업이 되지 않고 버려짐)" 상태입니다. 깃허브에 이슈를 올려도 원작자가 답변도 안 달아줍니다…

이 tool은 Microsoft에서 개발한 [DMTK parameter server (Multiverso)](https://github.com/Microsoft/Multiverso.git)라는 분산처리용 프레임워크를 기반으로 하고 있습니다(Multiverso는 다시 MPI 기반입니다).

그런데 이 DMWE가 참조하고 있는 Multiverso 버전이 `Multiverso-initial`이라는 branch인데 이것의 Readme를 보시면 1년 전에 이미 "deprecated" 되었다고 적혀 
있습니다. 그래서 DMWE를 사용하려면 최신 버전이 아닌 1년 전의 `Multiverso-initial`를 사용해야만 하고, 이 버전은 부분부분 기능이 없거나 에러가 존재합니다. 

개인적으로 최신 버전 Multiverso를 설치해 DMWE의 코드를 수정해서 써 보려고 했으나 한두 군데를 바꿔서 쓸 수 있는 것이 아니었습니다. 거의 구조를 통째로 바꿔야 하는 상황이라 최신 버전 적용은 포기하는 것이 좋을 듯합니다.

DMWE는 실행 쪽 코드는 python, 알고리즘 쪽 코드는 C++로 작성되어 있습니다. 이는 실행의 속도를 높이기 위해서라고 논문에서 밝히고 있습니다.


다운로드 (Download) 
----------
DMWE 코드를 다운로드합니다. 원본 다운로드 경로는 아래와 같습니다.

> `$ git clone https://github.com/Microsoft/distributed_skipgram_mixture`


그러나 아래에서 언급하겠지만 이 원본 코드로는 제대로 실행이 안됩니다. 제가 실행 파일과 멀티쓰레딩이 가능한 상태로 살짝 수정한 버전을 다운로드 하시려면 아래 경로로 다운로드 해주세요. 만약 멀티쓰레딩이 필요가 없으시면 `scripts/run.py`만 따로 받아서 바꾸셔도 됩니다.

> `$ git clone https://github.com/lynn-hong/distributed_skipgram_mixture`

아래 ‘실행’ 섹션에서 반드시 수정된 부분이 어디인지 확인하시고 본인에게 맞는 것을 다운로드 하시기 바랍니다.


이를 실행시키기 위한 프레임워크인 Multiverso를 DMWE 디렉토리 안으로 다운로드합니다. 앞서 말씀드린 대로 master branch가 아닙니다. 위치는 
`distributed_skipgram_mixture/multiverso`가 되어야 합니다.

> `$ git clone -b multiverso-initial https://github.com/Microsoft/multiverso.git`


빌드 (Build)
----------


실행 (Run)
----------
변수 세팅을 위해서는 `scripts/parameters_settings.txt`를 참고하시고, 실행은 `scripts/run.py`로 합니다.

원래 기본으로 들어 있는 `run.py`는 다양한 오류가 나면서 실행이 되지 않습니다…;; 여러 곳을 수정해야 하는데 제가 수정한 상태로 업데이트해 두었으니 그것을 쓰셔도 됩니다. 수정한 부분은 아래와 같습니다.

- `sg_mixture_args`에서 arg 숫자 0부터 24까지로 재조정

- `sg_mixture_args`에서 `-port` 파라미터 삭제

- `threads`를 1로 조정 (멀티쓰레딩 관련 사항은 아래 ‘이슈’ 파트에서 좀 더 자세히 설명하겠습니다. 제 버전 코드를 받으신 분들은 2 이상으로 바꾸셔도 됩니다.)

- 가장 마지막 줄의 MPI 실행 부분에서 경로를 `mpiexec.hydra`의 절대경로로 바꿈


파라미터 설명 (Description of parameters)
----------
`run.py`에서 설정하는 파라미터들은 `scripts/parameters_settings.txt`에 설명되어 있습니다. 모두 한국어로 번역하면서 누락된 설명을 추가해 보강했습니다. 아래는 각 파라미터를 설정하는 데 도움이 될 코멘트입니다.

`scripts/run.py` 등에서 경로를 설정할 때 full path로 적는 것을 권장해 드립니다. MPI 분산처리를 하는 경우에 특히 경로 문제가 많이 발생했습니다. 골치가 아프니 처음부터 절대경로로 쓰시는 것이 좋습니다


-train_file

-read_sense

- size: 

- window:

- threads:

- min_count:

- epoch:

- sense_num_multi:

- top_n:

- data_block_size

아래는 파라미터는 아니지만 경로 등을 수정해 주어야 하는 부분입니다.

- mpiexec 안의 distributed_skipgram_mixture를 bin 디렉토리 내의 excutable 파일 경로로 대체



결과 파일 설명 (Result files)
----------
- binary_embedding_file

- text_embedding_file

- huff_tree_file

- outputlayer_binary_file

- outputlayer_text_file


이슈 (Issues)
----------


추가 제언 (Additional comments)
----------

- 여러 대의 컴퓨터에 설치 및 실행을 해본 결과, 같은 리눅스 환경의 서버라도 설정 과정에서 오류가 발생하는 부분이 모두 달랐습니다. 본 문서에 적혀 있지 않은 이슈들이 발생할 가능성이 많으므로 혹시 해결이 잘 안되신다면 제보해 주시기 바랍니다!



Reference
----------
[1] Tian, F., Dai, H., Bian, J., Gao, B., Zhang, R., Chen, E., & Liu, T. Y. (2014). [A probabilistic model for learning multi-prototype word embeddings](http://www.aclweb.org/anthology/C14-1016). In Proceedings of COLING (pp. 151-160).

[2] Mikolov, T., Sutskever, I., Chen, K., Corrado, G. S., & Dean, J. (2013). Distributed representations of words and phrases and their compositionality. In Advances in neural information processing systems (pp. 3111-3119).

[3] 홍수린. (2017). 딥 러닝과 한국어 사전을 이용한 단어 의미 중의성 해소. 석사학위논문, 연세대학교, 서울.


Microsoft Open Source Code of Conduct
------------
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
