# LatamXIX: Spanish Newspaper Corpus from 19th Century Latin America with LLM OCR Correction

**Authors:** Tony Montes, Rubén Manrique, Laura Manrique, Arturo Rodriguez

## Running the Azure layout and OCR model

1. Copy the dataset in the NEWSPAPERS folder, the dataset must be folders with PDF files or image files
2. Run `PDF_preprocessing.ipynb`, The output folder data will be populated with the processed files.
3. Run `Layout_OCR.ipynb`

## Previous Steps

First, a _raw_ version of the dataset is on a `/raw` folder within the `/data` folder. This folder contains all the folders with the JSON files extracted from the Azure's OCR for each Newspaper; for example, the folder `/data/raw/PD168_El_oso_results` contains a set of JSON files with the format:

```json
{
	"metadata": {
		"id": "PD168",
		"newspaper": "El oso",
		"year": 1845,
		"city": "Lima, Perú",
		"file": "1",
		"page": "page_0"
	},
	"contexts": [
		{
			"id": 0,
			"text": "La publicacion del Oso se harà dos veces cada se mana, y constará de un pliego en cuarto ; ofreciendo à mas sus redactores, dar los gravados oportunos, siempre que loexija el asuntode que trate. Redactado por un Num. 8. TEMA del Periodico. POLITICA MILITAR. OCTAVA SESION. Abierta la sesion á las dore y un minuto de la noche , 25 de Febrero de 1845 , con asistencia de todos los Señores Representantes, se leyó y aprobó la acta de la Asamblea anterior , ménos en lo tocante à la torre del Convento de Santo Domingo, punto que quedó para ventilarse en mejor ocasion. En seguida se dió cuenta de una nota del Ejecutivo , referente à que urjía la necesidad de organizar un Ejército ; pues decia el Excmo. Decano: - \"Un poder sin bayonetas vale tanto como un cero puesto á la izquierda.\" ",
			"bounding_box": [43.0, 159.0, 496.0, 1496.0],
			"center": [269.5, 827.5]
		}
	]
}
```

## Steps

### 1. Structuring

The first step is to structure all the JSON files within a parquet file, this is done with the [`structuring.py`](./structuring.py) file, that reads all the JSONs and structures them.

### 2. Cleaning

The cleaning step is done in the [`cleaning.ipynb`](./cleaning.ipynb) notebook, which contains step by step all the substeps:

1. Remove duplicates and empty texts.
2. Filter out rows where 50%+ of the characters are non-alphabetic.
3. Remove all the rows with 4 or less tokens.

Also, an initial version of a [tokenizer](./data/tokenizer/) (trained from the [BETO](https://huggingface.co/dccuchile/bert-base-spanish-wwm-cased) pre-trained tokenizer in Spanish) is saved in this step

### 3. LLM OCR Correction

Finally, for the OCR correction with LLM there are 3 notebooks that must be ran in order:

1. [`get-llm-responses.ipynb`](./correcting/get-llm-responses.ipynb): Get the LLM responses from the correction requests for each text in the cleaned dataset.
2. [`get-corrections.ipynb`](./correcting/get-corrections.ipynb): From the LLM responses, find each correction made individually in order to later classify it between OCR error, surface form or none. Also, in this step, an initial OCR-correction detection is done for the most basic case.
3. [`classify-corrections.ipynb`](./correcting/classify-corrections.ipynb): Classify the corrections made by the LLM in the previous step, based on hardcoded rules and exceptions.
