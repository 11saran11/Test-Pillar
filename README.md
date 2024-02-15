# Test-Pillar

const createScriptXML = (configObject) => {
    const configObjectArray = JSON.parse(configObject);
    let currentPageId, currentPageElement, currentLoopElement, currentLoopTag, tempElement = null;

    readFile('./wordHTML.html', 'utf-8', (error, htmlCode) => {
        if (error) throw new Error(error);
        const parsedHtml = new JSDOM(htmlCode).window.document;
        const documentElement = Array.from(parsedHtml.querySelector('.WordSection1').children);
        if (documentElement)
            documentElement.forEach((element, index) => {
                if (element && element.classList) {
                    const styleProp = isStyleExist(configObjectArray, element.classList.toString());
                    if (styleProp) {
                        if (currentPageElement == null) {
                            currentPageElement = xmlDoc.createElement(styleProp.renderTag);
                            if (styleProp.style == 'pageId') {
                                currentPageId = element.textContent.replace(/\s+/g, ' ').replace(/\n/g, '').trim();
                                currentPageElement.setAttribute(styleProp.attributeName, currentPageId);
                                currentLoopElement = currentPageElement;
                                currentLoopTag = styleProp.renderTag;
                                return;
                            }
                        } else {

                            if (styleProp.style == 'pageId') {
                                currentPageElement.appendChild(currentLoopElement);
                                xmlDoc.documentElement.appendChild(currentPageElement);

                                currentPageElement = xmlDoc.createElement(styleProp.renderTag);
                                currentPageId = element.textContent.replace(/\s+/g, ' ').replace(/\n/g, '').trim();
                                currentPageElement.setAttribute(styleProp.attributeName, currentPageId);
                                currentLoopElement = currentPageElement;
                                currentLoopTag = styleProp.renderTag;
                                return;
                            }

                            if (currentLoopElement == null) {
                                currentLoopElement = xmlDoc.createElement(styleProp.renderTag);
                                currentLoopTag = styleProp.renderTag;
                            }

                            if (currentLoopTag == styleProp.renderTag) {
                                if (styleProp.isComponent) {
                                    if (styleProp.isChild) {
                                        const childElement = xmlDoc.createElement(styleProp.renderTag);
                                        if (styleProp.attributeName) {
                                            childElement.setAttribute(styleProp.attributeName, element.textContent.replace(/\s+/g, ' ').replace(/\n/g, '').trim());
                                        }

                                        if (currentLoopElement) {
                                            if (tempElement) {
                                                tempElement.appendChild(currentLoopElement);
                                                currentLoopElement = childElement;
                                            } else {
                                                currentLoopElement.appendChild(childElement);
                                            }
                                        }
                                    }
                                    else {
                                        const childElement = xmlDoc.createElement(styleProp.renderTag);
                                        if (styleProp.attributeName) {
                                            childElement.setAttribute(styleProp.attributeName, element.textContent.replace(/\s+/g, ' ').replace(/\n/g, '').trim());
                                        }

                                        currentLoopElement = childElement;
                                        currentPageElement.appendChild(childElement);
                                    }
                                } else if (styleProp.isAttribute) {
                                    currentLoopElement.setAttribute(styleProp.attributeName, element.textContent.replace(/\s+/g, ' ').replace(/\n/g, '').trim());
                                }
                            } else {
                                if (tempElement) {
                                    currentPageElement.appendChild(tempElement);
                                }

                                if (styleProp.targetTag && styleProp.isComponent) {
                                    if (tempElement)
                                        currentPageElement.appendChild(tempElement);
                                    tempElement = currentLoopElement;
                                }

                                currentLoopElement = xmlDoc.createElement(styleProp.renderTag);
                                if (styleProp.attributeName) {
                                    currentLoopElement.setAttribute(styleProp.attributeName, element.textContent.replace(/\s+/g, ' ').replace(/\n/g, '').trim());
                                }
                                currentLoopTag = styleProp.renderTag;
                            }

                        }
                    }
                }

                if (index == documentElement.length - 1) {
                    if (currentLoopElement) {
                        tempElement.appendChild(currentLoopElement);
                    }

                    if (tempElement) {
                        currentPageElement.appendChild(tempElement);
                    } else {
                        currentPageElement.appendChild(currentLoopElement);
                    }
                    xmlDoc.documentElement.appendChild(currentPageElement);
                }
            });

        const xmlString = new XMLSerializer().serializeToString(xmlDoc);
        writeFile('./script.xml', xmlFormat(xmlString), (error) => {
            if (error) throw new Error(error)
            console.log("File Generated Successfully");
        })


    });
}
