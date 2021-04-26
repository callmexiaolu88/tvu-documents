Microsoft.AspNetCore.Mvc.ApplicationModels.DefaultApplicationModelProvider.CanonicalizeActionName

private string CanonicalizeActionName(string actionName)
{
	if (_mvcOptions.SuppressAsyncSuffixInActionNames && actionName.EndsWith("Async", StringComparison.Ordinal))
	{
		actionName = actionName.Substring(0, actionName.Length - "Async".Length);
	}
	return actionName;
}

public MvcOptions()
{
	EnableEndpointRouting = true;
	SuppressAsyncSuffixInActionNames = true;
	MaxIAsyncEnumerableBufferLimit = 8192;
	base..ctor();
	CacheProfiles = new Dictionary<string, CacheProfile>(StringComparer.OrdinalIgnoreCase);
	Conventions = new List<IApplicationModelConvention>();
	Filters = new FilterCollection();
	FormatterMappings = new FormatterMappings();
	InputFormatters = new FormatterCollection<IInputFormatter>();
	OutputFormatters = new FormatterCollection<IOutputFormatter>();
	ModelBinderProviders = new List<IModelBinderProvider>();
	ModelBindingMessageProvider = new DefaultModelBindingMessageProvider();
	ModelMetadataDetailsProviders = new List<IMetadataDetailsProvider>();
	ModelValidatorProviders = new List<IModelValidatorProvider>();
	ValueProviderFactories = new List<IValueProviderFactory>();
}