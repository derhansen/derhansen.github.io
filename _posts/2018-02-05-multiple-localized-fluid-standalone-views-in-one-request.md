---
layout: post
title: TYPO3 - How to render a Fluid standalone view multiple times in different languages
date: '2018-02-05T14:12:00.000+01:00'
author: Torben Hansen
tags:
- fluid standalone view
- TYPO3
- localization
modified_time: '2018-02-05T14:12:25.310+01:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-8087346133079574866
blogger_orig_url: http://www.derhansen.de/2018/02/multiple-localized-fluid-standalone-views-in-one-request.html
permalink: /2018/02/multiple-localized-fluid-standalone-views-in-one-request.html
---

Back in 2015, I wrote 2
blogposts ([first](https://www.derhansen.de/2015/10/fluid-standaloneview-translation-scheduler-task.html)
and [second](https://www.derhansen.de/2015/11/typo3-using-fluid-standaloneview-to.html)) about rendering a localized
Fluid standalone view in a scheduler task (commandController). The main problem was to render a 
**Fluid standalone view** multiple times within the **same request** but with **different languages**. Back then, 
my solution was to create an own ViewHelper and a modified version of the TYPO3 LocalizationUtility which were 
responsible for handling the localization changes during the rendering request.

Meanwhile, I got feedback from readers of my blog pointing me to a **more simple solution** for the problem.

The main problem with changing the TYPO3 backend language during one request is the **language cache**, which is only
initialized once for the current language. So when you switch the backend language multiple times, the _cached language
files_ for the previous language will still be used.

The solution is to **unset the language cache** for the extension you are rendering your Fluid standalone view from. In
order to do so, you have to extend the TYPO3 LocalizationUtility with a new function as shown below.

{% highlight php %}
class LocalizationUtility extends \TYPO3\CMS\Extbase\Utility\LocalizationUtility
{
    /**
     * Resets the language cache for the given extension key
     *
     * @param string $extensionName
     */
    public static function resetLocalizationCache($extensionName)
    {
        unset(static::$LOCAL_LANG[$extensionName]);
    }
}
{% endhighlight %}

The example code below shows, how to use the _resetLocalizationCache_ method before rendering a Fluid standalone view in
a given language.

{% highlight php %}
/**
 * Renders a Fluid StandaloneView respecting the given language
 *
 * @param string $language The language (e.g. de, dk or se)
 * @return string
 */
public function renderStandaloneView($language = '')
{
    // Set the extensionKey
    $extensionKey = GeneralUtility::underscoredToUpperCamelCase('standaloneview');

    if ($language !== '') {
        // Temporary set Language of current BE user to given language
        $GLOBALS['BE_USER']->uc['lang'] = $language;
        LocalizationUtility::resetLocalizationCache($extensionKey);
    }

    /** @var \TYPO3\CMS\Fluid\View\StandaloneView $view */
    $view = $this->objectManager->get(StandaloneView::class);
    $view->setFormat('html');
    $template = GeneralUtility::getFileAbsFileName(
        'EXT:standaloneview/Resources/Private/Templates/StandaloneView.html'
    );
    $view->setTemplatePathAndFilename($template);

    // Set Extension name, so localizations for extension get respected
    $view->getRequest()->setControllerExtensionName($extensionKey);

    return $view->render();
}
{% endhighlight %}

So with a small extension of the LocalizationUtility it is now easily possible to render a Fluid standalone view in a
language of choice and it is also possible to switch the language within the same request (e.g. sending out localized
e-mails to multiple recipients).

I updated the [Demo-Extension](https://github.com/derhansen/standaloneview), which contains a backend module and a
command controller for demonstration purposes.

I would like to thank **Johannes Rebhan** for giving me the hint about the localization cache and also **Ulrich
Fischer** for showing me a [different approach](https://gist.github.com/derhansen/1eee357e472be595692bf655a28a1557),
which requires more code, but lead to the same result.