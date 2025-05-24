# Таксономическая классификация секвенированных микроорганизмов с помощью наивного байессовского классификатора и фреймворка Qiime2.
Содержание: 
1. Предварительная обработка данных;
1. Создание классификатора и обучение классификатора;
1. Классификация исходных последовательностей и обработка результата.

# 1. Предварительная обработка данных.
Работа с фреймворком требует перевода исходных данных в файлы артефактов. Для каждого файла необходимо корректно выбрать semantic type.

[Ссылка на документацию qiime2 по semantic type.](https://docs.qiime2.org/2022.11/semantic-types/)

- Для последовательностей секвеннированных организмов, таксономию которых будем определять, импортирование будет выглядеть следующим образом:

      qiime tools import \
      --input-path OTUs_uparse.fa \
      --output-path rep-seqs.qza \
      --type FeatureData[Sequence]

Для обучения классификатора по рекомендации была взята база *RDP 16S rRNA No18*

[Ссылка на RDPs sourceforge server, откуда была взята база.](https://sourceforge.net/projects/rdp-classifier/files/RDP_Classifier_TrainingData/)

К сожалению, хоть база данных и помечена как QiimeFormat, но Qiime2 не работает с последовательностями в нижнем регистре(возможно, это связано с разными версиями фреймворка).
Для Linux решение проблемы записывается следующим образом: 

`tr 'acgtrykmswbdhvn' 'ACGTRYKMSWBDHVN' < RefOTUs.fa > RefOTUs_uppercase.fa`

- Для последовательностей микроорганизмов из тренировочного датасета импорт будет выглядеть следующим образом:

      qiime tools import \
      --input-path RefOTUs_uppercase.fa \
      --output-path RefOTUs.qza \
      --type FeatureData[Sequence]

- Наилучшая классификация наивным байесовским классификатором достигается при обучении классификатора только на той области целевых последовательностей, которая была секвенирована. Извлечем эти данные из последовательностей при помощи праймеров:
        
      qiime feature-classifier extract-reads \
      --i-sequences RefOTUs.qza \
      --p-f-primer CTCCTACGGRRSGCAGCAG \
      --p-r-primer GGACTACNVGGGTWTCTAAT \
      --o-reads RefSeqs.qza

- Для таксономии микроорганизмов из тренировочного датасета импорт будет выглядеть следующим образом:

      qiime tools import \
      --input-path Ref_taxonomy.txt \
      --output-path RefTaxonomy.qza \
      --type FeatureData[Taxonomy] \
      --input-format HeaderlessTSVTaxonomyFormat

# 2. Создание и обучение классификатора.
- Создание таксономического классификатора происходит следующим образом:

      qiime feature-classifier fit-classifier-naive-bayes \
      --i-reference-reads RefSeqs.qza \
      --i-reference-taxonomy RefTaxonomy.qza \
      --o-classifier classifier.qza

- Обучение таксономического классификатора происходит следующим образом:

      qiime feature-classifier classify-sklearn 
      --i-classifier classifier.qza 
      --i-reads rep-seqs.qza 
      --o-classification taxonomy_result.qza

# 3. Классификация исходных последовательностей и обработка результата.
- Классификация проводится следующим образом:

      qiime feature-classifier classify-sklearn \
      --i-classifier classifier.qza \
      --i-reads rep-seqs.qza \
      --o-classification taxonomy_result.qza

- Для визуализации полученный результат нужно преобразовать в другой формат:

      qiime metadata tabulate \
      --m-input-file taxonomy_result.qza \
      --o-visualization taxonomy_result.qzv
 
Полученный результат можно [посмотреть на сайте qiime2 в разделе visualization.](https://view.qiime2.org)

Для удобства в репозиторий уже загружена таблица с результатами: [metadata.csv](https://github.com/confect1on/qiime_taxonomy_classification/files/10336175/metadata.csv)
