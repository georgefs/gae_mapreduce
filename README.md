google appengine 筆記
=============

input reader base

```python
class InputReader(model.JsonMixin):
  """Abstract base class for input readers.

  InputReaders have the following properties:
   * They are created by using the split_input method to generate a set of
     InputReaders from a MapperSpec.
   * They generate inputs to the mapper via the iterator interface.
   * After creation, they can be serialized and resumed using the JsonMixin
     interface.
   * They are cast to string for a user-readable description; it may be
     valuable to implement __str__.
  """

  # When expand_parameters is False, then value yielded by reader is passed
  # to handler as is. If it's true, then *value is passed, expanding arguments
  # and letting handler be a multi-parameter function.
  expand_parameters = False

  # Mapreduce parameters.
  _APP_PARAM = "_app"
  NAMESPACE_PARAM = "namespace"
  NAMESPACES_PARAM = "namespaces"  # Obsolete.

  def __iter__(self):
    return self

  def next(self):
    """Returns the next input from this input reader as a key, value pair.

    Returns:
      The next input from this input reader.
    """
    raise NotImplementedError("next() not implemented in %s" % self.__class__)

  @classmethod
  def from_json(cls, input_shard_state):
    """Creates an instance of the InputReader for the given input shard state.

    Args:
      input_shard_state: The InputReader state as a dict-like object.

    Returns:
      An instance of the InputReader configured using the values of json.
    """
    raise NotImplementedError("from_json() not implemented in %s" % cls)

  def to_json(self):
    """Returns an input shard state for the remaining inputs.

    Returns:
      A json-izable version of the remaining InputReader.
    """
    raise NotImplementedError("to_json() not implemented in %s" %
                              self.__class__)

  @classmethod
  def split_input(cls, mapper_spec):
    """Returns a list of input readers.

    This method creates a list of input readers, each for one shard.
    It attempts to split inputs among readers evenly.

    Args:
      mapper_spec: model.MapperSpec specifies the inputs and additional
        parameters to define the behavior of input readers.

    Returns:
      A list of InputReaders. None when no input data can be found.
    """
    raise NotImplementedError("split_input() not implemented in %s" % cls)

  @classmethod
  def validate(cls, mapper_spec):
    """Validates mapper spec and all mapper parameters.

    Input reader parameters are expected to be passed as "input_reader"
    subdictionary in mapper_spec.params.

    Pre 1.6.4 API mixes input reader parameters with all other parameters. Thus
    to be compatible, input reader check mapper_spec.params as well and
    issue a warning if "input_reader" subdicationary is not present.

    Args:
      mapper_spec: The MapperSpec for this InputReader.

    Raises:
      BadReaderParamsError: required parameters are missing or invalid.
    """
    if mapper_spec.input_reader_class() != cls:
      raise BadReaderParamsError("Input reader class mismatch")
 ```
 
 /mapreduce/status.py 80~152
 ```python
 
class MapReduceYaml(validation.Validated):
  """Root class for mapreduce.yaml.

  File format:

  mapreduce:
  - name: <mapreduce_name>
    mapper:
      - input_reader: google.appengine.ext.mapreduce.DatastoreInputReader
      - handler: path_to_my.MapperFunction
      - params:
        - name: foo
          default: bar
        - name: blah
          default: stuff
      - params_validator: path_to_my.ValidatorFunction

  Where
    mapreduce_name: The name of the mapreduce. Used for UI purposes.
    mapper_handler_spec: Full <module_name>.<function_name/class_name> of
      mapper handler. See MapreduceSpec class documentation for full handler
      specification.
    input_reader: Full <module_name>.<function_name/class_name> of the
      InputReader sub-class to use for the mapper job.
    params: A list of optional parameter names and optional default values
      that may be supplied or overridden by the user running the job.
    params_validator is full <module_name>.<function_name/class_name> of
      a callable to validate the mapper_params after they are input by the
      user running the job.
  """

  ATTRIBUTES = {
      "mapreduce": validation.Optional(validation.Repeated(MapreduceInfo))
  }

  @staticmethod
  def to_dict(mapreduce_yaml):
    """Converts a MapReduceYaml file into a JSON-encodable dictionary.

    For use in user-visible UI and internal methods for interfacing with
    user code (like param validation). as a list

    Args:
      mapreduce_yaml: The Pyton representation of the mapreduce.yaml document.

    Returns:
      A list of configuration dictionaries.
    """
    all_configs = []
    for config in mapreduce_yaml.mapreduce:
      out = {
          "name": config.name,
          "mapper_input_reader": config.mapper.input_reader,
          "mapper_handler": config.mapper.handler,
      }
      if config.mapper.params_validator:
        out["mapper_params_validator"] = config.mapper.params_validator
      if config.mapper.params:
        param_defaults = {}
        for param in config.mapper.params:
          param_defaults[param.name] = param.default or param.value
        out["mapper_params"] = param_defaults
      if config.params:
        param_defaults = {}
        for param in config.params:
          param_defaults[param.name] = param.default or param.value
        out["params"] = param_defaults
      if config.mapper.output_writer:
        out["mapper_output_writer"] = config.mapper.output_writer
      all_configs.append(out)

    return all_configs
```
 
 
 
