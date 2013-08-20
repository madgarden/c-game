Recently, I've been using a bitfield to represent the structure of my vertex data (a la Direct3D's [FVF](http://msdn.microsoft.com/en-us/library/windows/desktop/bb172559\(v=vs.85\).aspx)).  It's simple, compact, and fast. Though, it is a lil' inflexible when it comes to component variations--but there's a hack for that!

The gist of it is as follows:

  1. Define your vertex components as a bitfield;
  2. Use [CTZ](http://en.wikipedia.org/wiki/Count_trailing_zeros) to determine the offest of various components and stride;
  3. and use the resulting offset table to create a ID3D11InputLayout.

---


Define your vertex components as a bitfield:

```
enum vertex_components {
  POSITION    = (1 << 0),
  COLOR0      = (1 << 1),
  COLOR1      = (1 << 2),
  COLOR2      = (1 << 3),
  COLOR3      = (1 << 4),
  COLOR4      = (1 << 5),
  COLOR5      = (1 << 6),
  COLOR6      = (1 << 7),
  COLOR7      = (1 << 8),
  TEXCOORD0   = (1 << 9),
  TEXCOORD1   = (1 << 10),
  TEXCOORD2   = (1 << 11),
  TEXCOORD3   = (1 << 12),
  TEXCOORD4   = (1 << 13),
  TEXCOORD5   = (1 << 14),
  TEXCOORD6   = (1 << 15),
  TEXCOORD7   = (1 << 16),
  NORMAL      = (1 << 17),
  TANGENT     = (1 << 18),
  BITANGENT   = (1 << 19),
  BONEINDICES = (1 << 20),
  BONEWEIGHTS = (1 << 21)
};
```


Use count trailing zeros to determine the offest of various components and the vertex stride.

```
size_t stride_from_vertex_components(uint32_t components) {
  static const size_t cs /* component size */ [32] = {
    12,
    16, 16, 16, 16, 16, 16, 16, 16,
    12, 12, 12, 12, 12, 12, 12, 12,
    12, 12, 12,
    4, 4,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };

  size_t size = 0;
  while (components) {
    size += cs[__builtin_ctz(components)];
    components &= (components - 1); }
  return size;
}

void offsets_from_vertex_components(uint32_t components,
  size_t offsets[32], size_t* num_of_components) {
  static const size_t cs /* component size */ [32] = {
    12,
    16, 16, 16, 16, 16, 16, 16, 16,
    12, 12, 12, 12, 12, 12, 12, 12,
    12, 12, 12,
    4, 4,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };

  size_t offset = 0;
  *num_of_components = 0;
  for (; components; *num_of_components += 1) {
    const size_t component = __builtin_ctz(components);
    offsets[component] = offset;
    offset += cs[component];
    components &= (components - 1);
  }
}
```


Finally, use the resulting offset table to create a ID3D11InputLayout.

```
ID3D11InputLayout* create_mapping_from_components(uint32_t input,
  uint32_t shader) {
  static const D3D11_INPUT_ELEMENT_DESC input_element_descriptors[] = {
    { "POSITION",     0, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        1, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        2, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        3, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        4, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        5, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        6, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "COLOR",        7, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     0, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     1, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     2, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     3, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     4, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     5, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     6, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TEXCOORD",     7, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "NORMAL",       0, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "TANGENT",      0, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "BINORMAL",     0, DXGI_FORMAT_R32G32B32_FLOAT,    0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "BLENDINDICES", 0, DXGI_FORMAT_R8G8B8A8_UINT,      0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
    { "BLENDWEIGHT",  0, DXGI_FORMAT_R8G8B8A8_UNORM,     0, D3D11_APPEND_ALIGNED_ELEMENT, D3D11_INPUT_PER_VERTEX_DATA, 0 },
  };

  if (shader & (~input))
    /* Failure case: the shader requires data that is undefined. */

  size_t num_of_input_elements = 0;
  size_t component_offsets[32] = { 0, };
  D3D11_INPUT_ELEMENT_DESC input_elements[32];

  offsets_from_vertex_components(input, offsets, &num_of_input_elements);

  for (num_of_input_elements = 0; shader; ++num_of_input_elements) {
    const uint32_t component = __builtin_ctz(shader);
    /* You could memcpy and patch instead... */
    input_elements[num_of_input_elements].SemanticName =
      input_element_descriptors[component].SemanticName;
    input_elements[num_of_input_elements].SemanticIndex =
      input_element_descriptors[component].SemanticIndex;
    input_elements[num_of_input_elements].Format =
      input_element_descriptors[component].Format;
    input_elements[num_of_input_elements].InputSlot =
      input_element_descriptors[component].InputSlot;
    input_elements[num_of_input_elements].AlignedByteOffset =
      component_offsets[component];
    input_elements[num_of_input_elements].InputSlotClass =
      input_element_descriptors[component].InputSlotClass;
    input_elements[num_of_input_elements].InstanceDataStepRate =
      input_element_descriptors[component].InstanceDataStepRate;
    shader &= (shader - 1);
  }

  ID3D11InputLayout* input_layout;
  D3D11VertexShader* shader = /* ... */;
  const HRESULT hr = /* device */->CreateInputLayout(
    &input_elements[0], num_of_input_elements,
    (const void*)/* shader bytecode */,
    /* len(shader bytecode) */,
    &input_layout);
  if (FAILED(hr))
    /* Failure case: the signature doesn't match the input layout :S */

  return input_layout;
}
```

Et voilà!
