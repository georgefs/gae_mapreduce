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
 
 
 
 
