# DisCo: A New Frontier in Complex Scene Image Generation

**Blog Writers**: Md. Rifat Rahman (1905094) ,  Rakib Kibria (1905098) , Arnob Saha Ankon (1905108)  
**Date**: January 07, 2025

## Overview

Generating realistic images from textual descriptions or layout conditions has made remarkable progress in recent years. Yet, when tasked with rendering intricate scenes involving numerous objects and their relationships, these approaches often falter. Challenges such as unclear spatial arrangements or non-spatial interactions limit the potential of traditional methods. To tackle these limitations, the paper **"Scene Graph Disentanglement and Composition for Generalizable Complex Image Generation"** introduces **DisCo** – a framework that leverages scene graphs to redefine the boundaries of image generation.

Scene graphs offer a structured representation where objects are nodes, and their relationships form the edges. This structured approach provides a more precise way to depict complex interactions compared to unstructured textual descriptions or layouts. DisCo advances the field of scene graph-to-image (SG2I) generation by addressing critical issues such as ambiguous alignment of relationships, limited diversity in outputs, and poor generalizability for edits or manipulations.

The implications of this research extend far beyond academic exploration, offering practical solutions in industries ranging from graphic design to virtual reality. By enhancing controllability and fidelity in image generation, DisCo sets a new standard for applications requiring intricate visual compositions.


![ss1](https://github.com/user-attachments/assets/50270d54-577f-4461-a84a-6c355a089f3a)


---

## Main Contributions

### 1. **Semantics-Layout Variational AutoEncoder (SL-VAE)**
DisCo introduces the SL-VAE, a novel module designed to disentangle scene graphs into two fundamental components: 
- **Spatial Layouts:** The arrangement of objects in a scene.
- **Interactive Semantics:** The relationships and interactions among objects.

Traditional SG2I methods typically rely on a one-to-one mapping between scene graphs and layouts, limiting their diversity. SL-VAE breaks this constraint by enabling a one-to-many mapping through Gaussian distribution sampling. This approach allows for the generation of multiple plausible layouts and semantics for a single input graph, enhancing both flexibility and realism.


![ss2](https://github.com/user-attachments/assets/61b6e7c4-0d92-4ebb-b422-02d024faf41e)


#### Textual Scene Graph Construction in SL-VAE

Textual Scene Graph Construction in the **Semantics-Layout Variational AutoEncoder (SL-VAE)** enriches scene graphs by integrating semantic and spatial information to create comprehensive node and edge embeddings.

#### i. Semantic Enrichment with CLIP Text Embeddings:

- Nodes (objects) and edges (relationships) are enhanced with **CLIP embeddings**, providing rich contextual meanings derived from textual labels.

#### ii. Spatial Encoding with Bounding Boxes:

- Spatial layouts of objects are represented using bounding box coordinates \((x, y, w, h)\), which are encoded into a feature space using **Fourier mapping** to capture both low- and high-frequency spatial features.

#### iii. Fusion of Semantic and Spatial Information:

- The embeddings from CLIP and spatial coordinates are concatenated to form **node** and **edge embeddings** that encode both the position and meaning of objects and their relationships.

This process ensures alignment between textual and visual data, allowing SL-VAE to handle complex scenes with precise spatial arrangements and meaningful interactions.


#### Graph Union Encoding in Semantics-Layout Variational AutoEncoder (SL-VAE)


#### i. Input: Nodes and Edges from Scene Graph
- **Nodes (objects)** are represented by embeddings $\( \mathbf{O}_i \)$, which encode semantic and spatial information.
- **Edges (relationships)** are represented by embeddings $\( \mathbf{E}_{ij} \)$, capturing relationships between objects.
- Initial embeddings are constructed using **Textual Scene Graph Construction** with CLIP text embeddings, bounding box information, and learnable embeddings.

#### ii. Graph Union Encoding with a Triplet-GCN
- Uses a **triplet-based Graph Convolutional Network (GCN)** to process node and edge embeddings, capturing both topological structure and object interactions.
- Embeddings for nodes, edges, and neighbors are updated iteratively using a triplet function.
- After \( L \) layers, the final node embeddings capture both spatial layout and non-spatial interactions.

#### iii. Parameterizing the Latent Space
- Final node embeddings are mapped to a Gaussian latent space using two MLPs to compute the mean $\( \mu \)$ and variance $\( \sigma \)$ of the distribution:
  
  $\mathbf{Z} \sim \mathcal{N}(\mu, \sigma)$

- A KL divergence loss is applied to ensure smoothness in the latent space:
  
  $L_{\text{union}} = \text{KL}(q(\mathbf{Z} | \mathbf{O}, \mathbf{E}) \parallel p(\mathbf{Z}))$



#### Disentangled Semantics-Layout Decoding in SL-VAE

The **Disentangled Semantics-Layout Decoding** in the **Semantics-Layout Variational AutoEncoder (SL-VAE)** reconstructs two distinct components from the latent space: **spatial layouts** and **interactive semantics**. This separation allows for diverse and coherent scene generation while maintaining the integrity of object relationships.

#### i. **Objective**
The decoder disentangles the latent space $\( \mathbf{Z} \)$ (from Graph Union Encoding) into:
- **Spatial Layouts** $\( \mathbf{L} \)$: Positions and bounding boxes of objects.
- **Interactive Semantics** $\( \mathbf{S} \)$: Relationships and attributes between objects.

#### ii. **Separate Decoders**
- **Layout Decoder** $\( D_l \)$: Reconstructs spatial arrangement and bounding box coordinates for each object. It minimizes reconstruction error between predicted and ground truth bounding boxes.
  
  Loss function:
  $L_{\text{layout}} = \frac{1}{N_o} \sum_{i=1}^{N_o} \left| \mathbf{b}_i - \hat{\mathbf{b}}_i \right|_1$

- **Semantic Decoder** $\( D_s \)$: Reconstructs object relationships and attributes. Outputs semantic embeddings $\( \mathbf{S} = \{\mathbf{s}_i\} \)$, which are later used in a diffusion model for image synthesis.

#### iii. **Sampling for Diverse Outputs**
During inference, the decoder samples from the Gaussian latent space $\( \mathbf{Z} \)$ to generate diverse layouts and semantic outputs:
$\hat{\mathbf{b}}_i, \hat{\mathbf{s}}_i \sim \mathcal{N}(\mu, \sigma)$

#### iv. **Combining Layout and Semantics**
Outputs from both decoders are fused at the object level:
$\mathbf{c}_i = \mathbf{s}_i \oplus F(\mathbf{b}_i)$
Where $\( F(\mathbf{b}_i) \)$ is the Fourier-mapped spatial embedding of the bounding box.

#### v. **Advantages of Disentanglement**
- **Independent Control**: Allows independent manipulation of object positions and relationships.
- **Diversity**: Generates diverse outputs by sampling different combinations of layout and semantics.
- **Accuracy**: Improves the precision of both spatial and semantic predictions, enhancing scene coherence.

#### vi. **Training the Decoders**
The decoders are trained jointly using the total loss function:
$L_{\text{total}} = \lambda_1 L_{\text{LDM}} + \lambda_2 L_{\text{union}} + \lambda_3 L_{\text{layout}}$

Where $\( L_{\text{LDM}} \)$ is the loss for the diffusion model, $\( L_{\text{union}} \)$ is the KL divergence loss, and $\( L_{\text{layout}} \)$ is the layout reconstruction loss.



### 2. **Compositional Masked Attention (CMA)**
To inject object-level information into the diffusion model, DisCo employs Compositional Masked Attention. This mechanism:
- Prevents semantic confusion by ensuring that relationships between objects are preserved.
- Mitigates attribute leakage through the use of object-specific attention masks.

By combining fine-grained attributes with global relational information, CMA significantly improves the rationality and controllability of the generated images. This is particularly critical in scenes with overlapping objects or intricate interactions.



#### 1. **Object-Level Fusion Tokenizer**
This component integrates spatial layouts and semantic embeddings into a unified representation for each object in a scene.
- **Input**: Includes spatial layout (bounding boxes), semantic embeddings (attributes and relationships), and optional additional attributes (e.g., color, size).
- **Fourier Mapping**: Spatial bounding box coordinates are transformed into a higher-dimensional space using Fourier mapping.
- **Fusion**: The spatial embedding and semantic embedding are concatenated to create an object-level embedding. Additional attributes can be fused into the embedding, and a null embedding is used for padding if fewer objects exist in the scene.
- **Output**: A set of object-level embeddings representing spatial, semantic, and attribute information for each object.

#### 2. **Compositional Masked Attention (CMA)**
CMA controls how tokens representing visual elements or objects interact in the diffusion process.
- **Attention Mask**: Ensures that tokens within the same object interact with each other while preventing interactions across different objects.
- **CMA Layer**: Applies the attention mask to the concatenated visual and object tokens, preserving object-specific relationships and preventing unwanted influence between objects.
- **Benefits**: Prevents relational confusion, avoids attribute leakage between objects, and encodes fine-grained details of object relationships and attributes into the diffusion model.

#### 3. **Diffusion Loss**
The Diffusion Loss ensures the model accurately reconstructs images conditioned on the input scene graph by comparing predicted and true noise during the denoising process.
- **Noise Prediction**: Noise is added to the latent representation at each timestep, and the model predicts the added noise.
- **Loss Function**: The model is trained to minimize the difference between the predicted noise and the true noise during denoising.
- **Conditioning with Graph Information**: The diffusion loss is conditioned on the object-level embeddings, allowing the model to incorporate spatial and semantic relationships while reconstructing the target image.





### 3. **Multi-Layered Sampler (MLS)**
One of DisCo’s standout features is its ability to edit and manipulate scene graphs while maintaining consistency in the resulting images. The Multi-Layered Sampler enables object-level graph manipulation, such as adding new nodes (objects) or modifying attributes. By leveraging diverse conditions generated by SL-VAE, MLS ensures that edits remain visually coherent with the original scene, making it ideal for dynamic content creation or iterative design workflows.



### 4. **Superior Performance**
Extensive evaluations on benchmark datasets like Visual Genome and COCO-Stuff showcase DisCo’s superiority over existing methods. The model achieves:
- Higher **Inception Scores (IS)**, reflecting better image quality and diversity.
- Lower **Fréchet Inception Distances (FID)**, indicating closer alignment with real image distributions.

In user studies, DisCo consistently outperformed competitors in terms of generation rationality and controllability, further cementing its practical viability.

---

## Insights and Implications

DisCo represents a paradigm shift in generative AI by emphasizing the disentanglement of layouts and semantics. This separation enables a richer understanding of scene graphs, leading to more accurate and diverse visual outputs. The innovations in SL-VAE and CMA not only enhance generation quality but also provide a robust foundation for future advancements.

### Broader Impacts

#### **Applications**
- **Creative Industries:** Graphic designers and artists can use DisCo to generate complex scene compositions, speeding up workflows and inspiring creativity.
- **Synthetic Data Generation:** AI practitioners can create diverse datasets for training machine learning models, particularly in object detection and relationship modeling.
- **Virtual Reality and Gaming:** DisCo’s ability to generate detailed, interactive environments makes it a valuable tool for immersive content creation.

#### **Future Directions**
While DisCo excels in generating static images, its principles can be extended to:
- **Video Generation:** Modeling temporal dynamics to produce consistent, realistic video frames.
- **3D Scene Generation:** Expanding from 2D layouts to 3D spatial representations for augmented reality applications.
- **Multi-Modal Integration:** Combining scene graphs with audio or text to create richer, multi-sensory experiences.

---

## Why This Matters

At its core, DisCo bridges the gap between structured representations and generative models, addressing long-standing challenges in image synthesis. By integrating scene graphs into the diffusion process, it offers an unprecedented level of control and fidelity. The implications for industries reliant on visual content creation are profound, enabling tools that are not only powerful but also intuitive and adaptable.

DisCo exemplifies how structured data and advanced generative models can work in harmony, setting a new benchmark for complex scene generation. As the field of AI continues to evolve, frameworks like DisCo pave the way for a future where machines can not only interpret but also create intricate visual narratives.
